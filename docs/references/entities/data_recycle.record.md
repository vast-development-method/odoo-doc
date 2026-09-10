# Recycling Record (`data_recycle.record`)

**Transport name:** `data_recycle.record`  
**Storage name:** `data_recycle_record`  
**Kind:** persistent entity (one table)  
**Defined by package:** `data_recycle`

Description: Recycling Record

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Record Name | single line text |  | computed by rule `_compute_name` (not stored) |
| `recycle_model_id` | Recycle Model | many to one | `data_recycle.model` | indexed (btree_not_null); on delete of the target: cascade |
| `res_id` | Record identifier | integer |  | indexed |
| `res_model_id` | Resource Model | many to one |  | read only; related through path `recycle_model_id.res_model_id` and stored |
| `res_model_name` | Resource Model Name | single line text |  | read only; related through path `recycle_model_id.res_model_name` and stored |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_company_id` | preparation rule | self, record | `data_recycle` | model |  |
| `_compute_name` | computation | self | `data_recycle` | depends: `res_id` |  |
| `_compute_company_id` | computation | self | `data_recycle` | depends: `res_id` |  |
| `_original_records` | internal rule | self | `data_recycle` |  |  |
| `action_validate` | user action | self | `data_recycle` |  |  |
| `action_discard` | user action | self | `data_recycle` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `data_recycle` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `data_recycle.view_data_recycle_record_list` | list |  | `active`, `res_model_name`, `res_id`, `recycle_model_id`, `name` | `Validate`, `Discard` |  | `data_recycle` |
| `data_recycle.view_data_recycle_record_search` | search |  | `recycle_model_id` |  | `Discarded` | `data_recycle` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `data_recycle.action_data_recycle_record` | Field Recycle Records | list,form |  |  |  | `data_recycle` |
| `data_recycle.action_data_recycle_record_notification` | Field Recycle Records | list,form |  | `{ 'searchpanel_default_recycle_model_id': active_id }` |  | `data_recycle` |

Machine-readable definition: `../../../schemas/data/entities/data_recycle.record.json`; views: `../../../schemas/interfaces/views/data_recycle.record.json`.
