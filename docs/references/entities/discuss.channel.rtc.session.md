# Mail RTC session (`discuss.channel.rtc.session`)

**Transport name:** `discuss.channel.rtc.session`  
**Storage name:** `discuss_channel_rtc_session`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `im_livechat`

Description: Mail RTC session

## Identity and behavior

- Mixins (classical inheritance): `bus.listener.mixin`
- Display name field: `channel_member_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `channel_member_id` | Channel Member | many to one | `discuss.channel.member` | required; on delete of the target: cascade |
| `channel_id` | Channel | many to one | `discuss.channel` | read only; related through path `channel_member_id.channel_id` and stored; indexed (btree_not_null) |
| `partner_id` | Partner | many to one | `res.partner` | related through path `channel_member_id.partner_id` and stored; indexed |
| `guest_id` | Guest | many to one | `mail.guest` | related through path `channel_member_id.guest_id` |
| `write_date` | Last Updated On | date and time |  | indexed |
| `is_screen_sharing_on` | Is sharing the screen | boolean |  |  |
| `is_camera_on` | Is sending user video | boolean |  |  |
| `is_muted` | Is microphone muted | boolean |  |  |
| `is_deaf` | Has disabled incoming sound | boolean |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_channel_member_unique` | Constraint | `UNIQUE(channel_member_id)` | There can only be one rtc session per channel member | `mail` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `im_livechat`, `mail` | model_create_multi |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `_bus_channel` | internal rule | self | `mail` |  |  |
| `_update_and_broadcast` | internal rule | self, values | `mail` |  | Updates the session and notifies all members of the channel of the change. |
| `_gc_inactive_sessions` | background operation | self | `mail` | autovacuum | Garbage collect sessions that aren't active anymore, this can happen when the server or the user's browser crash or when the user's system session ends. |
| `action_disconnect` | user action | self | `mail` |  |  |
| `_delete_inactive_rtc_sessions` | internal rule | self | `mail` |  | Deletes the inactive sessions from self. |
| `_notify_peers` | internal rule | self, notifications | `mail` |  | Used for peer-to-peer communication, guarantees that the sender is the current guest or partner.  :param notifications: list of tuple with the following elements:     - target_session_ids: a list of discuss.channel.rtc.session ids     - content: a string with the content to be sent to the targets |
| `_to_store_defaults` | internal rule | self, target | `mail` |  |  |
| `_get_store_extra_fields` | preparation rule | self | `mail` |  |  |
| `_inactive_rtc_session_domain` | internal rule | self | `mail` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.discuss_channel_rtc_session_view_search` | search |  | `channel_member_id` |  | `Channel` | `mail` |
| `mail.discuss_channel_rtc_session_view_tree` | list |  | `id`, `channel_member_id`, `channel_id`, `write_date` | `Disconnect` |  | `mail` |
| `mail.discuss_channel_rtc_session_view_form` | form |  | `channel_member_id`, `channel_id`, `partner_id`, `guest_id`, `is_screen_sharing_on`, `is_camera_on`, `is_muted`, `is_deaf` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.discuss_channel_rtc_session_action` | RTC sessions | list,form |  | `{'search_default_group_by_channel': True}` |  | `mail` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mail.discuss_channel_rtc_session_menu` | RTC sessions | `mail.mail_menu_technical` | `mail.discuss_channel_rtc_session_action` | 52 |  |

Machine-readable definition: `../../../schemas/data/entities/discuss.channel.rtc.session.json`; views: `../../../schemas/interfaces/views/discuss.channel.rtc.session.json`.
