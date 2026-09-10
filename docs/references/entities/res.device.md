# Devices (`res.device`)

**Transport name:** `res.device`  
**Storage name:** `res_device`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Devices

## Identity and behavior

- Mixins (classical inheritance): `res.device.log`
- Default ordering: `last_activity desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `revoke` | operation | self | `base` |  |  |
| `_revoke` | internal rule | self | `base` |  |  |
| `_select` | internal rule | self | `base` | model |  |
| `_from` | internal rule | self | `base` | model |  |
| `_where` | internal rule | self | `base` | model |  |
| `_query` | internal rule | self | `base` |  |  |
| `init` | lifecycle override | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Users can read only their own devices | `[Command.link(ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Administrators can read all devices | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.res_device_view_form` | form |  | `user_id`, `display_name`, `ip_address`, `first_activity`, `last_activity`, `linked_ip_addresses` |  |  | `base` |
| `base.res_device_view_tree` | list |  | `user_id`, `display_name`, `ip_address`, `first_activity`, `last_activity`, `country`, `city` | `Revoke` |  | `base` |
| `base.res_device_view_kanban` | kanban |  | `id`, `device_type`, `last_activity`, `is_current`, `display_name`, `ip_address`, `country`, `city` | `Log out` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_user_device` | User Devices | list,kanban,form |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/res.device.json`; views: `../../../schemas/interfaces/views/res.device.json`.
