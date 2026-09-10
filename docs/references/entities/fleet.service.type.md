# Fleet Service Type (`fleet.service.type`)

**Transport name:** `fleet.service.type`  
**Storage name:** `fleet_service_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`

Description: Fleet Service Type

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `category` | Category | selection |  | required; Help: Choose whether the service refer to contracts, vehicle services or both |

## Selection values

### `category` (Category)

| Value | Label |
|---|---|
| `contract` | Contract |
| `service` | Service |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_service_types_view_tree` | list |  | `name`, `category` |  |  | `fleet` |
| `fleet.fleet_vehicle_service_types_view_search` | search |  | `name`, `category` |  | `groupby_category` | `fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_service_types_action` | Types | list,form |  | `{"search_default_groupby_category" : True}` |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.service.type.json`; views: `../../../schemas/interfaces/views/fleet.service.type.json`.
