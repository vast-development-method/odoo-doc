# Group of dashboards (`spreadsheet.dashboard.group`)

**Transport name:** `spreadsheet.dashboard.group`  
**Storage name:** `spreadsheet_dashboard_group`  
**Kind:** persistent entity (one table)  
**Defined by package:** `spreadsheet_dashboard`

Description: Group of dashboards

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `dashboard_ids` | Dashboard | one to many | `spreadsheet.dashboard` | inverse field `dashboard_group_id` |
| `published_dashboard_ids` | Published Dashboard | one to many | `spreadsheet.dashboard` | restricted by domain `[["is_published", "=", true]]`; inverse field `dashboard_group_id` |
| `sequence` | Sequence | integer |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_unlink_except_spreadsheet_data` | internal rule | self | `spreadsheet_dashboard` | ondelete |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_spreadsheet_data` | UserError | You cannot delete %s as it is used in another module. | `spreadsheet_dashboard` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.group_dashboard_manager` | yes | yes | yes | yes | `spreadsheet_dashboard` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `spreadsheet_dashboard.spreadsheet_dashboard_container_view_list` | list |  | `sequence`, `name` |  |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.spreadsheet_dashboard_container_view_form` | form |  | `name`, `dashboard_ids` |  |  | `spreadsheet_dashboard` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `spreadsheet_dashboard.spreadsheet_dashboard_action_configuration_dashboards` | Dashboards | list,form |  |  |  | `spreadsheet_dashboard` |

Machine-readable definition: `../../../schemas/data/entities/spreadsheet.dashboard.group.json`; views: `../../../schemas/interfaces/views/spreadsheet.dashboard.group.json`.
