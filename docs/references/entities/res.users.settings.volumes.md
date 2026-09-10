# User Settings Volumes (`res.users.settings.volumes`)

**Transport name:** `res.users.settings.volumes`  
**Storage name:** `res_users_settings_volumes`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: User Settings Volumes

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_setting_id` | User Setting | many to one | `res.users.settings` | required; indexed; on delete of the target: cascade |
| `partner_id` | Partner | many to one | `res.partner` | indexed; on delete of the target: cascade |
| `guest_id` | Guest | many to one | `res.partner` | indexed; on delete of the target: cascade |
| `volume` | Volume | float |  | default `0.5`; Help: Ranges between 0.0 and 1.0, scale depends on the browser implementation |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_partner_unique` | UniqueIndex | `(user_setting_id, partner_id) WHERE partner_id IS NOT NULL` |  | `mail` |
| `_guest_unique` | UniqueIndex | `(user_setting_id, guest_id) WHERE guest_id IS NOT NULL` |  | `mail` |
| `_partner_or_guest_exists` | Constraint | `CHECK((partner_id IS NOT NULL AND guest_id IS NULL) OR (partner_id IS NULL AND guest_id IS NOT NULL))` | A volume setting must have a partner or a guest. | `mail` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `mail` | depends: `user_setting_id`, `partner_id`, `guest_id` |  |
| `_discuss_users_settings_volume_format` | internal rule | self | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| res.users.settings.volumes: access their own entries | `[Command.link(ref('base.group_user'))]` | `[('user_setting_id.user_id', '=', user.id)]` | True | True | True | True |
| Administrators can access all User Settings volumes. | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/res.users.settings.volumes.json`.
