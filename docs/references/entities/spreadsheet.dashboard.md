# Spreadsheet Dashboard (`spreadsheet.dashboard`)

**Transport name:** `spreadsheet.dashboard`  
**Storage name:** `spreadsheet_dashboard`  
**Kind:** persistent entity (one table)  
**Defined by package:** `spreadsheet_dashboard`

Description: Spreadsheet Dashboard

## Identity and behavior

- Mixins (classical inheritance): `spreadsheet.mixin`
- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `dashboard_group_id` | Dashboard Group | many to one | `spreadsheet.dashboard.group` | required; indexed |
| `sequence` | Sequence | integer |  |  |
| `sample_dashboard_file_path` | Sample Dashboard File Path | single line text |  |  |
| `is_published` | Is Published | boolean |  | default `True` |
| `company_ids` | Companies | many to many | `res.company` |  |
| `group_ids` | Group | many to many | `res.groups` | default computed dynamically (lambda self: self.env.ref('base.group_user')) |
| `favorite_user_ids` | Favorite Users | many to many | `res.users` | restricted by domain `lambda self: [('id', '=', self.env.uid)]`; Help: Users who have favorited this dashboard |
| `is_favorite` | Is Favorite | boolean |  | computed by rule `_compute_is_favorite` (not stored); Help: Indicates whether the dashboard is favorited by the current user |
| `main_data_model_ids` | Main Data Model | many to many | `ir.model` | not copied on duplication |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_favorite` | computation | self | `spreadsheet_dashboard` | depends_context: `uid`; depends: `favorite_user_ids` |  |
| `action_toggle_favorite` | user action | self | `spreadsheet_dashboard` |  |  |
| `_get_serialized_readonly_dashboard` | preparation rule | self | `spreadsheet_dashboard` |  |  |
| `_get_sample_dashboard` | preparation rule | self | `spreadsheet_dashboard` |  |  |
| `_dashboard_is_empty` | internal rule | self | `spreadsheet_dashboard` |  |  |
| `_get_dashboard_translation_namespace` | preparation rule | self | `spreadsheet_dashboard` |  |  |
| `copy_data` | lifecycle override | self, default | `spreadsheet_dashboard` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.group_dashboard_manager` | yes | yes | yes | yes | `spreadsheet_dashboard` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Spreadsheet dashboard: groups | `[(4, ref('base.group_user'))]` | `[('group_ids', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| Dashboard multi-company | global (all users) | `[('company_ids', 'in', company_ids + [False])]` | True | True | True | True |
| Spreadsheet dashboard: manager | `[(4, ref('spreadsheet_dashboard.group_dashboard_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `spreadsheet_dashboard.spreadsheet_dashboard_view_list` | list |  | `sequence`, `name`, `group_ids`, `company_ids`, `spreadsheet_binary_data`, `is_published`, `dashboard_group_id` |  |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.spreadsheet_dashboard_view_form` | form |  | `name`, `dashboard_group_id`, `company_ids`, `group_ids`, `spreadsheet_binary_data` |  |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.spreadsheet_dashboard_view_kanban` | kanban |  | `name` |  |  | `spreadsheet_dashboard` |

Machine-readable definition: `../../../schemas/data/entities/spreadsheet.dashboard.json`; views: `../../../schemas/interfaces/views/spreadsheet.dashboard.json`.
