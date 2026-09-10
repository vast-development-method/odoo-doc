# Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. (`res.role`)

**Transport name:** `res.role`  
**Storage name:** `res_role`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users.

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `user_ids` | Users | many to many | `res.users` | association table `res_role_res_users_rel` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_name` | UniqueIndex | `(name)` | A role with the same name already exists. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `mail` |
| `base.group_erp_manager` | yes | yes | yes | yes | `mail` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.res_role_view_search` | search |  | `name`, `user_ids` |  | `My Roles`, `Users` | `mail` |
| `mail.res_role_view_tree` | list |  | `name`, `user_ids` |  |  | `mail` |
| `mail.res_role_view_form` | form |  | `name`, `user_ids` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.res_role_action` | Roles | list,form |  |  |  | `mail` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mail.menu_roles` | Roles | `mail.menu_configuration` | `mail.res_role_action` | 25 |  |

Machine-readable definition: `../../../schemas/data/entities/res.role.json`; views: `../../../schemas/interfaces/views/res.role.json`.
