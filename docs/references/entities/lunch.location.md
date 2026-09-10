# Lunch Locations (`lunch.location`)

**Transport name:** `lunch.location`  
**Storage name:** `lunch_location`  
**Kind:** persistent entity (one table)  
**Defined by package:** `lunch`

Description: Lunch Locations

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Location Name | single line text |  | required |
| `address` | Address | multi line text |  |  |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_lunch_user` | no | yes | yes | no | `lunch` |
| `group_lunch_manager` | yes | yes | yes | yes | `lunch` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Lunch location: Multi Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_location_view_search` | search |  | `name`, `address` |  |  | `lunch` |
| `lunch.lunch_location_form_view` | form |  | `name`, `address`, `company_id` |  |  | `lunch` |
| `lunch.lunch_location_tree_view` | list |  | `name`, `address`, `company_id` |  |  | `lunch` |
| `lunch.lunch_location_kanban_view` | kanban |  | `name`, `company_id`, `address` |  |  | `lunch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_location_action` | Lunch Locations | list,form,kanban |  |  |  | `lunch` |

Machine-readable definition: `../../../schemas/data/entities/lunch.location.json`; views: `../../../schemas/interfaces/views/lunch.location.json`.
