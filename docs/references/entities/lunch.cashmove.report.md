# Cashmoves report (`lunch.cashmove.report`)

**Transport name:** `lunch.cashmove.report`  
**Storage name:** `lunch_cashmove_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `lunch`

Description: Cashmoves report

## Identity and behavior

- Default ordering: `date desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `id` | identifier | identifier |  |  |
| `amount` | Amount | float |  |  |
| `date` | Date | date |  |  |
| `currency_id` | Currency | many to one | `res.currency` |  |
| `user_id` | User | many to one | `res.users` |  |
| `description` | Description | multi line text |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `lunch` |  |  |
| `init` | lifecycle override | self | `lunch` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `lunch` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_cashmove_report_view_search` | search |  | `description`, `user_id` |  | `Payment`, `My Account grouped`, `By User` | `lunch` |
| `lunch.lunch_cashmove_report_view_search_2` | search |  | `description`, `user_id` |  | `By Employee` | `lunch` |
| `lunch.lunch_cashmove_report_view_tree` | list |  | `currency_id`, `date`, `user_id`, `description`, `amount` |  |  | `lunch` |
| `lunch.lunch_cashmove_report_view_tree_2` | list |  | `currency_id`, `date`, `description`, `amount` |  |  | `lunch` |
| `lunch.lunch_cashmove_report_view_form` | form |  | `currency_id`, `user_id`, `date`, `amount`, `description` |  |  | `lunch` |
| `lunch.view_lunch_cashmove_report_kanban` | kanban |  | `currency_id`, `description`, `amount`, `date`, `user_id` |  |  | `lunch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_cashmove_report_action_account` | My Account | list | `[('user_id','=',uid)]` |  |  | `lunch` |
| `lunch.lunch_cashmove_report_action_control_accounts` | Control Accounts | list,kanban,form |  | `{"search_default_group_by_user":1}` |  | `lunch` |

Machine-readable definition: `../../../schemas/data/entities/lunch.cashmove.report.json`; views: `../../../schemas/interfaces/views/lunch.cashmove.report.json`.
