# Model of a vehicle (`fleet.vehicle.model`)

**Transport name:** `fleet.vehicle.model`  
**Storage name:** `fleet_vehicle_model`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`

Description: Model of a vehicle

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `avatar.mixin`
- Default ordering: `name asc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (27)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Model name | single line text |  | required; changes are tracked in the message thread |
| `brand_id` | Manufacturer | many to one | `fleet.vehicle.model.brand` | required; changes are tracked in the message thread; indexed (btree_not_null) |
| `category_id` | Category | many to one | `fleet.vehicle.model.category` | changes are tracked in the message thread |
| `vendors` | Vendors | many to many | `res.partner` | association table `fleet_vehicle_model_vendors` |
| `image_128` | Image 128 | image |  | read only; related through path `brand_id.image_128` |
| `active` | Active | boolean |  | default `True` |
| `vehicle_type` | Vehicle Type | selection |  | required; default `car`; changes are tracked in the message thread |
| `transmission` | Transmission | selection |  | changes are tracked in the message thread |
| `vehicle_count` | Vehicle Count | integer |  | computed by rule `_compute_vehicle_count` (not stored); searchable through a search rule |
| `model_year` | Model Year | selection |  | changes are tracked in the message thread; values provided by rule `_get_year_selection` |
| `color` | Color | single line text |  | changes are tracked in the message thread |
| `seats` | Seating Capacity | integer |  | changes are tracked in the message thread |
| `doors` | Number of Doors | integer |  | changes are tracked in the message thread; Help: Specifies the total number of doors, including the truck and hatch doors, if applicable. |
| `trailer_hook` | Trailer Hitch | boolean |  | default ; changes are tracked in the message thread; Help: A trailer hitch is a device attached to a vehicle's chassis for towing purposes,            such as pulling trailers, boats, or other vehicles. |
| `default_co2` | CO₂ Emissions | float |  | changes are tracked in the message thread |
| `co2_emission_unit` | Co2 Emission Unit | selection |  | required; computed by rule `_compute_co2_emission_unit` (not stored) |
| `co2_standard` | Emission Standard | single line text |  | changes are tracked in the message thread; Help: Emission Standard specifies the regulatory test procedure or             guideline under which a vehicle's emissions are measured. |
| `default_fuel_type` | Fuel Type | selection |  | default `electric`; changes are tracked in the message thread |
| `power` | Power | float |  | changes are tracked in the message thread |
| `horsepower` | Horsepower | float |  | changes are tracked in the message thread |
| `horsepower_tax` | Horsepower Taxation | float |  | changes are tracked in the message thread |
| `electric_assistance` | Electric Assistance | boolean |  | default ; changes are tracked in the message thread |
| `power_unit` | Power Unit | selection |  | required; default `power` |
| `vehicle_properties_definition` | Vehicle Properties | properties definition |  |  |
| `vehicle_range` | Range | integer |  |  |
| `range_unit` | Range Unit | selection |  | required; default `km` |
| `drive_type` | Drive Type | selection |  |  |

## Selection values

### `vehicle_type` (Vehicle Type)

| Value | Label |
|---|---|
| `car` | Car |
| `bike` | Bike |

### `transmission` (Transmission)

| Value | Label |
|---|---|
| `manual` | Manual |
| `automatic` | Automatic |

### `co2_emission_unit` (Co2 Emission Unit)

| Value | Label |
|---|---|
| `g/km` | g/km |
| `g/mi` | g/mi |

### `power_unit` (Power Unit)

| Value | Label |
|---|---|
| `power` | kW |
| `horsepower` | Horsepower (hp) |

### `range_unit` (Range Unit)

| Value | Label |
|---|---|
| `km` | km |
| `mi` | mi |

### `drive_type` (Drive Type)

| Value | Label |
|---|---|
| `fwd` | Front-Wheel Drive (FWD) |
| `awd` | All-Wheel Drive (AWD) |
| `rwd` | Rear-Wheel Drive (RWD) |
| `4wd` | Four-Wheel Drive (4WD) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_year_selection` | preparation rule | self | `fleet` |  |  |
| `_search_display_name` | search rule | self, operator, value | `fleet` | model |  |
| `_compute_display_name` | computation | self | `fleet` | depends: `brand_id` |  |
| `_compute_vehicle_count` | computation | self | `fleet` |  |  |
| `_compute_co2_emission_unit` | computation | self | `fleet` | depends: `range_unit` |  |
| `_search_vehicle_count` | search rule | self, operator, value | `fleet` | model |  |
| `action_model_vehicle` | user action | self | `fleet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_model_view_form` | form |  | `vehicle_count`, `image_128`, `name`, `brand_id`, `active`, `vehicle_type`, `category_id`, `model_year`, `seats`, `doors`, `color`, `trailer_hook`, `electric_assistance`, `default_fuel_type`, `transmission`, `drive_type`, `power`, `power_unit`, `vehicle_range`, `range_unit`, `default_co2`, `co2_emission_unit`, `co2_standard`, `horsepower`, `power_unit`, `horsepower_tax`, `vendors`, `name`, `phone`, `email` | `action_model_vehicle` |  | `fleet` |
| `fleet.fleet_vehicle_model_view_tree` | list |  | `brand_id`, `name`, `vehicle_count`, `category_id`, `vehicle_type`, `default_co2` |  |  | `fleet` |
| `fleet.fleet_vehicle_model_view_kanban` | kanban |  | `name`, `brand_id` |  |  | `fleet` |
| `fleet.fleet_vehicle_model_view_search` | search |  | `name`, `brand_id` |  | `Contains Vehicle`, `Archived`, `Manufacturers`, `Category`, `Vehicle Type` | `fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_model_action` | Models | list,form |  | `{"search_default_groupby_brand" : True,}` |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.model.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.model.json`.
