# Vehicle (`fleet.vehicle`)

**Transport name:** `fleet.vehicle`  
**Storage name:** `fleet_vehicle`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`  
**Extended by packages:** `account_fleet`, `hr_fleet`

Description: Vehicle

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `avatar.mixin`
- Default ordering: `license_plate asc, acquisition_date asc`
- Display name search fields: `["name", "driver_id.name"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (69)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_vehicle_name` and stored |
| `description` | Vehicle Description | rich text |  |  |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread |
| `manager_id` | Fleet Manager | many to one | `res.users` | restricted by domain `lambda self: f"[('share', '=', False), ('company_id', '=', company_id), ('all_group_ids', 'in', {self.env.ref('fleet.fleet_group_user').id})]"` |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `country_id` | Country | many to one | `res.country` | related through path `company_id.country_id` |
| `country_code` | Country Code | single line text |  | related through path `country_id.code` |
| `license_plate` | License Plate | single line text |  | changes are tracked in the message thread; Help: License plate number of the vehicle (i = plate number for a car) |
| `vin_sn` | Chassis Number | single line text |  | changes are tracked in the message thread; not copied on duplication; Help: Unique number written on the vehicle motor (VIN/SN number) |
| `trailer_hook` | Trailer Hitch | boolean |  | computed by rule `_compute_trailer_hook` and stored; default ; Help: A trailer hitch is a device attached to a vehicle's chassis for towing purposes,             such as pulling trailers, boats, or other vehicles. |
| `driver_id` | Driver | many to one | `res.partner` | changes are tracked in the message thread; not copied on duplication; Help: Driver address of the vehicle |
| `future_driver_id` | Future Driver | many to one | `res.partner` | changes are tracked in the message thread; not copied on duplication; must belong to the same company; Help: Next Driver Address of the vehicle |
| `model_id` | Model | many to one | `fleet.vehicle.model` | required; changes are tracked in the message thread |
| `brand_id` | Brand | many to one | `fleet.vehicle.model.brand` | related through path `model_id.brand_id` and stored |
| `log_drivers` | Assignment Logs | one to many | `fleet.vehicle.assignation.log` | inverse field `vehicle_id` |
| `log_services` | Services Logs | one to many | `fleet.vehicle.log.services` | inverse field `vehicle_id` |
| `log_contracts` | Contracts | one to many | `fleet.vehicle.log.contract` | inverse field `vehicle_id` |
| `contract_count` | Contract Count | integer |  | computed by rule `_compute_count_all` (not stored) |
| `service_count` | Services | integer |  | computed by rule `_compute_count_all` (not stored) |
| `odometer_count` | Odometer | integer |  | computed by rule `_compute_count_all` (not stored) |
| `history_count` | Drivers History Count | integer |  | computed by rule `_compute_count_all` (not stored) |
| `next_assignation_date` | Assignment Date | date |  | Help: This is the date at which the car will be available, if not set it means available instantly |
| `order_date` | Order Date | date |  |  |
| `acquisition_date` | Registration Date | date |  | default computed dynamically (fields.Date.today); changes are tracked in the message thread; Help: Date of vehicle registration |
| `write_off_date` | Cancellation Date | date |  | changes are tracked in the message thread; Help: Date when the vehicle's license plate has been cancelled/removed. |
| `contract_date_start` | First Contract Date | date |  | default computed dynamically (fields.Date.today); changes are tracked in the message thread |
| `color` | Color | single line text |  | computed by rule `_compute_color` and stored; Help: Color of the vehicle |
| `state_id` | State | many to one | `fleet.vehicle.state` | default computed dynamically (_get_default_state); changes are tracked in the message thread; on delete of the target: set null; Help: Current state of the vehicle |
| `location` | Location | single line text |  | Help: Location of the vehicle (garage, ...) |
| `seats` | Seating Capacity | integer |  | computed by rule `_compute_seats` and stored; Help: Number of seats of the vehicle |
| `model_year` | Model Year | selection |  | computed by rule `_compute_model_year` and stored; values provided by rule `_get_year_selection`; Help: Year of the model |
| `doors` | Number of Doors | integer |  | computed by rule `_compute_doors` and stored; Help: Number of doors of the vehicle |
| `tag_ids` | Tags | many to many | `fleet.vehicle.tag` | not copied on duplication; association table `fleet_vehicle_vehicle_tag_rel` |
| `odometer` | Last Odometer | float |  | computed by rule `_get_odometer` (not stored); writable through an inverse rule; Help: Odometer measure of the vehicle at the moment of this log |
| `odometer_unit` | Odometer Unit | selection |  | required; default `kilometers` |
| `transmission` | Transmission | selection |  | computed by rule `_compute_transmission` and stored |
| `fuel_type` | Fuel Type | selection |  | computed by rule `_compute_fuel_type` and stored |
| `power_unit` | Power Unit | selection |  | required; default `power` |
| `horsepower` | Horsepower | float |  | computed by rule `_compute_horsepower` and stored |
| `horsepower_tax` | Horsepower Taxation | float |  | computed by rule `_compute_horsepower_tax` and stored |
| `power` | Power | float |  | computed by rule `_compute_power` and stored; Help: Power in kW of the vehicle |
| `co2` | CO₂ Emissions | float |  | computed by rule `_compute_co2` and stored; changes are tracked in the message thread; Help: CO2 emissions of the vehicle |
| `co2_emission_unit` | Co2 Emission Unit | selection |  | required; computed by rule `_compute_co2_emission_unit` and stored; default `g/km` |
| `co2_standard` | Emission Standard | single line text |  | computed by rule `_compute_co2_standard` and stored; Help: Emission Standard specifies the regulatory test procedure             or guideline under which a vehicle's emissions are measured. |
| `category_id` | Category | many to one | `fleet.vehicle.model.category` | computed by rule `_compute_category` and stored |
| `image_128` | Image 128 | image |  | read only; related through path `model_id.image_128` |
| `contract_renewal_due_soon` | Has Contracts to renew | boolean |  | computed by rule `_compute_contract_reminder` (not stored); searchable through a search rule |
| `contract_renewal_overdue` | Has Contracts Overdue | boolean |  | computed by rule `_compute_contract_reminder` (not stored); searchable through a search rule |
| `contract_state` | Last Contract State | selection |  | computed by rule `_compute_contract_reminder` (not stored) |
| `car_value` | Catalog Value (value-added tax Incl.) | float |  | changes are tracked in the message thread |
| `net_car_value` | Purchase Value | float |  |  |
| `residual_value` | Residual Value | float |  |  |
| `plan_to_change_car` | Plan To Change Car | boolean |  | changes are tracked in the message thread |
| `plan_to_change_bike` | Plan To Change Bike | boolean |  | changes are tracked in the message thread |
| `vehicle_type` | Vehicle Type | selection |  | related through path `model_id.vehicle_type` |
| `frame_type` | Bike Frame Type | selection |  |  |
| `electric_assistance` | Electric Assistance | boolean |  | computed by rule `_compute_electric_assistance` and stored |
| `frame_size` | Frame Size | float |  |  |
| `service_activity` | Service Activity | selection |  | computed by rule `_compute_service_activity` (not stored) |
| `vehicle_properties` | Properties | properties |  |  |
| `vehicle_range` | Range | integer |  |  |
| `range_unit` | Range Unit | selection |  | required; computed by rule `_compute_range_unit` and stored; default `km` |
| `bill_count` | Bills Count | integer |  | computed by rule `_compute_move_ids` (not stored) |
| `account_move_ids` | Account Move | one to many | `account.move` | computed by rule `_compute_move_ids` (not stored) |
| `mobility_card` | Mobility Card | single line text |  | computed by rule `_compute_mobility_card` and stored |
| `driver_employee_id` | Driver (Employee) | many to one | `hr.employee` | computed by rule `_compute_driver_employee_id` and stored; changes are tracked in the message thread; indexed (btree_not_null); restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `driver_employee_name` | Driver Employee Name | single line text |  | related through path `driver_employee_id.name` |
| `future_driver_employee_id` | Future Driver (Employee) | many to one | `hr.employee` | computed by rule `_compute_future_driver_employee_id` and stored; changes are tracked in the message thread; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |

## Selection values

### `odometer_unit` (Odometer Unit)

| Value | Label |
|---|---|
| `kilometers` | km |
| `miles` | mi |

### `transmission` (Transmission)

| Value | Label |
|---|---|
| `manual` | Manual |
| `automatic` | Automatic |

### `power_unit` (Power Unit)

| Value | Label |
|---|---|
| `power` | kW |
| `horsepower` | Horsepower |

### `co2_emission_unit` (Co2 Emission Unit)

| Value | Label |
|---|---|
| `g/km` | g/km |
| `g/mi` | g/mi |

### `contract_state` (Last Contract State)

| Value | Label |
|---|---|
| `futur` | Incoming |
| `open` | In Progress |
| `expired` | Expired |
| `closed` | Closed |

### `frame_type` (Bike Frame Type)

| Value | Label |
|---|---|
| `diamant` | Diamant |
| `trapez` | Trapez |
| `wave` | Wave |

### `service_activity` (Service Activity)

| Value | Label |
|---|---|
| `none` | None |
| `overdue` | Overdue |
| `today` | Today |

### `range_unit` (Range Unit)

| Value | Label |
|---|---|
| `km` | km |
| `mi` | mi |

## State fields

State machine fields of this entity: `contract_state`. Transitions are specified in the domain documents.

## Operations (47)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_state` | preparation rule | self | `fleet` |  |  |
| `_get_year_selection` | preparation rule | self | `fleet` |  |  |
| `_compute_service_activity` | computation | self | `fleet` | depends: `log_services` |  |
| `_load_fields_from_model` | internal rule | self, fields_to_load | `fleet` |  | Copies the desired fields from the models to the vehicles |
| `_compute_category` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_range_unit` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_trailer_hook` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_vehicle_range` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_electric_assistance` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_co2_standard` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_co2` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_power` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_horsepower` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_horsepower_tax` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_fuel_type` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_transmission` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_doors` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_model_year` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_seats` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_color` | computation | self | `fleet` | depends: `model_id` |  |
| `_compute_vehicle_name` | computation | self | `fleet` | depends: `model_id.brand_id.name`, `model_id.name`, `license_plate` |  |
| `_compute_co2_emission_unit` | computation | self | `fleet` | depends: `range_unit` |  |
| `_get_odometer` | preparation rule | self | `fleet` |  |  |
| `_set_odometer` | internal rule | self | `fleet` |  |  |
| `_compute_count_all` | computation | self | `fleet` |  |  |
| `_compute_contract_reminder` | computation | self | `fleet` | depends: `log_contracts` |  |
| `_get_analytic_name` | preparation rule | self | `fleet` |  |  |
| `_search_contract_renewal_due_soon` | search rule | self, operator, value | `fleet` |  |  |
| `_search_get_overdue_contract_reminder` | search rule | self, operator, value | `fleet` |  |  |
| `create` | lifecycle override | self, vals_list | `fleet`, `hr_fleet` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `fleet`, `hr_fleet` |  |  |
| `_get_driver_history_data` | preparation rule | self, vals | `fleet` |  |  |
| `create_driver_history` | operation | self, vals | `fleet` |  |  |
| `action_accept_driver_change` | user action | self | `fleet` |  |  |
| `return_action_to_open` | operation | self | `fleet` |  | This opens the xml view specified in xml_id for the current vehicle |
| `act_show_log_cost` | operation | self | `fleet` |  | This opens log view to view and add new log for this vehicle, groupby default to only show effective costs @return: the costs log view |
| `_track_subtype` | messaging hook | self, init_values | `fleet` |  |  |
| `open_assignation_logs` | operation | self | `fleet`, `hr_fleet` |  |  |
| `action_send_email` | user action | self | `fleet` |  |  |
| `action_open_odometer_report` | user action | self | `fleet` |  |  |
| `_compute_move_ids` | computation | self | `account_fleet` |  |  |
| `action_view_bills` | user action | self | `account_fleet` |  |  |
| `_compute_driver_employee_id` | computation | self | `hr_fleet` | depends: `driver_id` |  |
| `_compute_future_driver_employee_id` | computation | self | `hr_fleet` | depends: `future_driver_id` |  |
| `_compute_mobility_card` | computation | self | `hr_fleet` | depends: `driver_id` |  |
| `_update_create_write_vals` | internal rule | self, vals | `hr_fleet` |  |  |
| `action_open_employee` | user action | self | `hr_fleet` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | The odometer value cannot be lower than the previous one. | `fleet` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | yes | yes | yes | yes | `fleet` |
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `hr.group_hr_user` | no | yes | no | no | `hr_fleet` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Administrator has all rights on vehicle | `[Command.link(ref('fleet_group_manager'))]` |  | True | True | True | True |
| Fleet vehicle: Multi Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Hr Officer read rights on vehicle with employees assigned | `[(4, ref('hr.group_hr_user'))]` | `['\|', ('driver_employee_id', '!=', False), ('future_driver_employee_id', '!=', False)]` | True | False | False | False |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_fleet.fleet_vehicle_view_form` | xpath | `fleet.fleet_vehicle_view_form` | `bill_count` | `action_view_bills` |  | `account_fleet` |
| `fleet.fleet_vehicle_view_form` | form |  | `service_activity`, `state_id`, `company_id`, `currency_id`, `country_code`, `history_count`, `contract_count`, `service_count`, `service_count`, `service_count`, `odometer_count`, `image_128`, `model_id`, `license_plate`, `tag_ids`, `active`, `vehicle_type`, `driver_id`, `future_driver_id`, `next_assignation_date`, `company_id`, `category_id`, `order_date`, `acquisition_date`, `write_off_date`, `vin_sn`, `odometer`, `odometer_unit`, `manager_id`, `location`, `plan_to_change_car`, `plan_to_change_bike`, `vehicle_properties`, `horsepower_tax`, `contract_date_start`, `car_value`, `net_car_value`, `residual_value`, `model_year`, `seats`, `doors`, `color`, `trailer_hook`, `frame_type`, `frame_size`, `electric_assistance`, `fuel_type`, `transmission`, `power`, `power_unit`, `horsepower`, `power_unit`, `vehicle_range`, `range_unit`, `co2`, `co2_emission_unit`, `co2_standard`, `description` | `Apply New Driver`, `open_assignation_logs`, `return_action_to_open`, `return_action_to_open`, `return_action_to_open`, `return_action_to_open`, `return_action_to_open`, `action_open_odometer_report` |  | `fleet` |
| `fleet.fleet_vehicle_view_tree` | list |  | `active`, `license_plate`, `model_id`, `category_id`, `manager_id`, `driver_id`, `future_driver_id`, `log_drivers`, `vin_sn`, `co2`, `acquisition_date`, `tag_ids`, `state_id`, `contract_renewal_due_soon`, `contract_renewal_overdue`, `vehicle_properties`, `contract_state`, `activity_exception_decoration` |  |  | `fleet` |
| `fleet.fleet_vehicle_view_search` | search |  | `name`, `log_drivers`, `model_id`, `license_plate`, `tag_ids`, `state_id`, `vehicle_properties` |  | `Available`, `Bikes`, `Cars`, `Trailer Hook`, `Planned for Change`, `Need Action`, `Archived`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Model`, `Brand`, `Status`, `Fuel Type`, `Properties` | `fleet` |
| `fleet.fleet_vehicle_view_form_quick_create` | form |  | `model_id`, `license_plate`, `tag_ids` |  |  | `fleet` |
| `fleet.fleet_vehicle_view_kanban` | kanban |  | `contract_renewal_due_soon`, `contract_renewal_overdue`, `image_128`, `license_plate`, `model_id`, `tag_ids`, `driver_id`, `driver_id`, `future_driver_id`, `location`, `vehicle_properties`, `contract_count`, `activity_ids` |  |  | `fleet` |
| `fleet.fleet_vehicle_view_activity` | activity |  | `license_plate`, `id`, `license_plate`, `model_id` |  |  | `fleet` |
| `fleet.fleet_vehicle_view_pivot` | pivot |  | `state_id`, `brand_id`, `model_id`, `license_plate` |  |  | `fleet` |
| `hr_fleet.fleet_vehicle_view_form_inherit_hr` | xpath | `fleet.fleet_vehicle_view_form` | `driver_employee_id`, `mobility_card` |  |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_view_search_inherit_hr` | xpath | `fleet.fleet_vehicle_view_search` | `mobility_card` |  |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_view_tree_inherit_hr` | field | `fleet.fleet_vehicle_view_tree` | `driver_id`, `driver_employee_id` |  |  | `hr_fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_action` | Vehicles | kanban,list,form,pivot,activity |  |  |  | `fleet` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `fleet.action_fleet_vehicle_send_mail` | Mail to Driver | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.json`.
