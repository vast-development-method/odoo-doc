# Device Log (`res.device.log`)

**Transport name:** `res.device.log`  
**Storage name:** `res_device_log`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Device Log

## Identity and behavior

- Display name search fields: `["platform", "browser"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `session_identifier` | Session Identifier | single line text |  | required; indexed (btree) |
| `platform` | Platform | single line text |  |  |
| `browser` | Browser | single line text |  |  |
| `ip_address` | internet protocol Address | single line text |  |  |
| `country` | Country | single line text |  |  |
| `city` | City | single line text |  |  |
| `device_type` | Device Type | selection |  |  |
| `user_id` | User | many to one | `res.users` | indexed (btree) |
| `first_activity` | First Activity | date and time |  |  |
| `last_activity` | Last Activity | date and time |  | indexed (btree) |
| `revoked` | Revoked | boolean |  | Help: If True, the session file corresponding to this device                                     no longer exists on the filesystem. |
| `is_current` | Current Device | boolean |  | computed by rule `_compute_is_current` (not stored) |
| `linked_ip_addresses` | Linked internet protocol address | multi line text |  | computed by rule `_compute_linked_ip_addresses` (not stored) |

## Selection values

### `device_type` (Device Type)

| Value | Label |
|---|---|
| `computer` | Computer |
| `mobile` | Mobile |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_composite_idx` | Index | `(user_id, session_identifier, platform, browser, last_activity, id) WHERE revoked IS NOT TRUE` |  | `base` |
| `_revoked_idx` | Index | `(revoked) WHERE revoked IS NOT TRUE` |  | `base` |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `base` |  |  |
| `_compute_is_current` | computation | self | `base` |  |  |
| `_compute_linked_ip_addresses` | computation | self | `base` |  |  |
| `_order_field_to_sql` | internal rule | self, alias, field_name, direction, nulls, query | `base` |  |  |
| `_is_mobile` | internal rule | self, platform | `base` |  |  |
| `_update_device` | internal rule | self, request | `base` | model | Must be called when we want to update the device for the current request. Passage through this method must leave a "trace" in the session.  :param request: Request or WebsocketRequest object |
| `_gc_device_log` | background operation | self | `base` | autovacuum |  |
| `__update_revoked` | internal rule | self | `base` | autovacuum | Set the field `revoked` to `True` for `res.device.log` for which the session file no longer exists on the filesystem. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Users can read only their own device logs | `[Command.link(ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Administrators can read all device logs | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/res.device.log.json`.
