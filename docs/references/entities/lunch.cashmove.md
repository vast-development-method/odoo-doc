# Lunch Cashmove (`lunch.cashmove`)

**Transport name:** `lunch.cashmove`  
**Storage name:** `lunch_cashmove`  
**Kind:** persistent entity (one table)  
**Defined by package:** `lunch`

Description: Lunch Cashmove

## Identity and behavior

- Default ordering: `date desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `currency_id` | Currency | many to one | `res.currency` | required; default computed dynamically (lambda self: self.env.company.currency_id) |
| `user_id` | User | many to one | `res.users` | default computed dynamically (lambda self: self.env.uid) |
| `date` | Date | date |  | required; default computed dynamically (fields.Date.context_today) |
| `amount` | Amount | float |  | required |
| `description` | Description | multi line text |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `lunch` |  |  |
| `get_wallet_balance` | operation | self, user, include_config | `lunch` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_lunch_user` | no | yes | no | no | `lunch` |
| `group_lunch_manager` | yes | yes | yes | yes | `lunch` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| lunch.cashmove: do not see other people's cashmove | `[(4, ref('group_lunch_user'))]` | `[('user_id', '=', user.id)]` | True | True | True | True |
| lunch.cashmove: do see other people's cashmove | `[(4, ref('group_lunch_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_cashmove_view_search` | search |  | `description`, `user_id` |  | `My Account grouped`, `By User` | `lunch` |
| `lunch.lunch_cashmove_view_tree` | list |  | `currency_id`, `date`, `user_id`, `description`, `amount` |  |  | `lunch` |
| `lunch.lunch_cashmove_view_form` | form |  | `currency_id`, `user_id`, `date`, `amount`, `description` |  |  | `lunch` |
| `lunch.view_lunch_cashmove_kanban` | kanban |  | `currency_id`, `description`, `amount`, `date`, `user_id` |  |  | `lunch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_cashmove_action_payment` | Cash Moves | list,kanban,form |  |  |  | `lunch` |

Machine-readable definition: `../../../schemas/data/entities/lunch.cashmove.json`; views: `../../../schemas/interfaces/views/lunch.cashmove.json`.
