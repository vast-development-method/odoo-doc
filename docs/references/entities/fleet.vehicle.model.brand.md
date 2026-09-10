# Brand of the vehicle (`fleet.vehicle.model.brand`)

**Transport name:** `fleet.vehicle.model.brand`  
**Storage name:** `fleet_vehicle_model_brand`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`

Description: Brand of the vehicle

## Identity and behavior

- Default ordering: `name asc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `active` | Active | boolean |  | default `True` |
| `image_128` | Logo | image |  |  |
| `model_count` |  | integer |  | computed by rule `_compute_model_count` and stored |
| `model_ids` | Model | one to many | `fleet.vehicle.model` | inverse field `brand_id` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_model_count` | computation | self | `fleet` | depends: `model_ids.active` |  |
| `action_brand_model` | user action | self | `fleet` |  |  |
| `action_open_brand_form` | user action | self | `fleet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_model_brand_view_tree` | list |  | `name`, `model_count` |  |  | `fleet` |
| `fleet.fleet_vehicle_model_brand_view_form` | form |  | `model_count`, `name`, `image_128` | `action_brand_model` |  | `fleet` |
| `fleet.fleet_vehicle_model_brand_view_kanban` | kanban |  | `active`, `image_128`, `name`, `model_count` |  |  | `fleet` |
| `fleet.fleet_vehicle_model_brand_view_search` | search |  | `name` |  | `With Models`, `Archived` | `fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_model_brand_action` | Manufacturers | kanban,list,form |  | `{'search_default_with_models': 1}` |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.model.brand.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.model.brand.json`.
