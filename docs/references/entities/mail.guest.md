# Guest (`mail.guest`)

**Transport name:** `mail.guest`  
**Storage name:** `mail_guest`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Guest

## Identity and behavior

- Mixins (classical inheritance): `avatar.mixin`, `bus.listener.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `access_token` | Access Token | single line text |  | required; read only; default computed dynamically (lambda self: str(uuid.uuid4())); not copied on duplication; visible only to groups `base.group_system` |
| `country_id` | Country | many to one | `res.country` |  |
| `email` | Email | single line text |  |  |
| `lang` | Language | selection |  |  |
| `timezone` | Timezone | selection |  |  |
| `channel_ids` | Channels | many to many | `discuss.channel` | not copied on duplication; association table `discuss_channel_member` |
| `presence_ids` | Presence | one to many | `mail.presence` | visible only to groups `base.group_system`; inverse field `guest_id` |
| `im_status` | IM Status | single line text |  | computed by rule `_compute_im_status` (not stored) |
| `offline_since` | Offline since | date and time |  | computed by rule `_compute_im_status` (not stored) |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_lang_get` | internal rule | self | `mail` | model |  |
| `_compute_im_status` | computation | self | `mail` | depends: `presence_ids.status` |  |
| `_get_guest_from_token` | preparation rule | self, token | `mail` |  | Returns the guest record for the given token, if applicable. |
| `_get_guest_from_context` | preparation rule | self | `mail` |  | Returns the current guest record from the context, if applicable. |
| `_get_or_create_guest` | preparation rule | self, guest_name, country_code, timezone | `mail` |  |  |
| `_get_timezone_from_request` | preparation rule | self, request | `mail` |  |  |
| `_update_name` | internal rule | self, name | `mail` |  |  |
| `_update_timezone` | internal rule | self, timezone | `mail` |  |  |
| `_get_im_status_access_token` | preparation rule | self | `mail` |  | Return a scoped access token for the `im_status` field. The token is used in `ir_websocket._prepare_subscribe_data` to grant access to presence channels.  :rtype: str |
| `_field_store_repr` | internal rule | self, field_name | `mail` |  |  |
| `_to_store_defaults` | internal rule | self, target | `mail` |  |  |
| `_set_auth_cookie` | internal rule | self | `mail` |  | Add a cookie to the response to identify the guest. Every route that expects a guest will make use of it to authenticate the guest through `add_guest_to_context`. |
| `_format_auth_cookie` | internal rule | self | `mail` |  | Format the cookie value for the given guest.  :return: formatted cookie value :rtype: str |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_update_name` | UserError | Guest's name cannot be empty. | `mail` |
| `_update_name` | UserError | Guest's name is too long. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `mail` |
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_guest_view_tree` | list |  | `id`, `name`, `country_id`, `lang`, `timezone` |  |  | `mail` |
| `mail.mail_guest_view_form` | form |  | `name`, `country_id`, `lang`, `timezone`, `channel_ids` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_guest_action` | Guests | list,form |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.guest.json`; views: `../../../schemas/interfaces/views/mail.guest.json`.
