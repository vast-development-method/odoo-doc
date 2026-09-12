# User Settings (`res.users.settings`)

**Transport name:** `res.users.settings`  
**Storage name:** `res_users_settings`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `bus`, `mail`, `calendar`, `im_livechat`, `google_calendar`, `project`, `microsoft_calendar`

Description: User Settings

## Identity and behavior

- Mixins (classical inheritance): `bus.listener.mixin`
- Display name field: `user_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | required; on delete of the target: cascade; restricted by domain `[["res_users_settings_id", "=", false]]` |
| `embedded_actions_config_ids` | Embedded Actions Config | one to many | `res.users.settings.embedded.action` | inverse field `user_setting_id` |
| `is_discuss_sidebar_category_channel_open` | Is discuss sidebar category channel open? | boolean |  | default `True` |
| `is_discuss_sidebar_category_chat_open` | Is discuss sidebar category chat open? | boolean |  | default `True` |
| `push_to_talk_key` | Push-To-Talk shortcut | single line text |  | Help: String formatted to represent a key with modifiers following this pattern: shift.ctrl.alt.key, e.g: truthy.1.true.b |
| `use_push_to_talk` | Use the push to talk feature | boolean |  | default  |
| `voice_active_duration` | Duration of voice activity in ms | integer |  | default `200`; Help: How long the audio broadcast will remain active after passing the volume threshold |
| `volume_settings_ids` | Volumes of other partners | one to many | `res.users.settings.volumes` | inverse field `user_setting_id` |
| `channel_notifications` | Channel Notifications | selection |  | Help: This setting will only be applied to channels. Mentions only if not specified. |
| `calendar_default_privacy` | Calendar Default Privacy | selection |  | required; default `public`; Help: Default privacy setting for whom the calendar events will be visible. |
| `livechat_username` | Livechat Username | single line text |  | Help: This username will be used as your name in the livechat channels. |
| `livechat_lang_ids` | Livechat languages | many to many | `res.lang` | Help: These languages, in addition to your main language, will be used to assign you to Live Chat sessions. |
| `livechat_expertise_ids` | Live Chat Expertise | many to many | `im_livechat.expertise` | Help: When forwarding live chat conversations, the chatbot will prioritize users with matching expertise. |
| `google_calendar_rtoken` | Refresh Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `google_calendar_token` | User token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `google_calendar_token_validity` | Token Validity | date and time |  | not copied on duplication; visible only to groups `base.group_system` |
| `google_calendar_sync_token` | Next Sync Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `google_calendar_cal_id` | Calendar identifier | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: Last Calendar ID who has been synchronized. If it is changed, we remove all links between GoogleID and the system Google Internal ID |
| `google_synchronization_stopped` | Google Synchronization stopped | boolean |  | not copied on duplication; visible only to groups `base.group_system` |
| `microsoft_calendar_sync_token` | Microsoft Next Sync Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `microsoft_synchronization_stopped` | Outlook Synchronization stopped | boolean |  | not copied on duplication; visible only to groups `base.group_system` |
| `microsoft_last_sync_date` | Last Sync Date | date and time |  | not copied on duplication; visible only to groups `base.group_system`; Help: Last synchronization date with Outlook Calendar |

## Selection values

### `channel_notifications` (Channel Notifications)

| Value | Label |
|---|---|
| `all` | All Messages |
| `no_notif` | Nothing |

### `calendar_default_privacy` (Calendar Default Privacy)

| Value | Label |
|---|---|
| `public` | Public |
| `private` | Private |
| `confidential` | Only internal users |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_user_id` | Constraint | `UNIQUE(user_id)` | One user should only have one user settings. | `base` |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_fields_blacklist` | preparation rule | self | `base`, `calendar`, `google_calendar`, `microsoft_calendar` | model | Get list of fields that won't be formatted. |
| `_find_or_create_for_user` | internal rule | self, user | `base` | model |  |
| `_res_users_settings_format` | internal rule | self, fields_to_format | `base` |  |  |
| `_format_settings` | internal rule | self, fields_to_format | `base`, `mail`, `web` | model |  |
| `set_res_users_settings` | operation | self, new_settings | `base`, `mail` |  |  |
| `get_embedded_actions_settings` | operation | self | `project`, `web` |  |  |
| `set_embedded_actions_setting` | operation | self, action_id, res_id, vals | `web` |  |  |
| `_bus_channel` | internal rule | self | `bus` |  |  |
| `set_volume_setting` | operation | self, partner_id, volume, guest_id | `mail` |  | Saves the volume of a guest or a partner. Either partner_id or guest_id must be specified. :param float volume: the selected volume between 0 and 1 :param int partner_id: :param int guest_id: |
| `_set_google_auth_tokens` | internal rule | self, access_token, refresh_token, ttl | `google_calendar` |  |  |
| `_google_calendar_authenticated` | internal rule | self | `google_calendar` |  |  |
| `_is_google_calendar_valid` | internal rule | self | `google_calendar` |  |  |
| `_refresh_google_calendar_token` | internal rule | self | `google_calendar` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_refresh_google_calendar_token` | UserError | error_msg | `google_calendar` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `base` |
| `group_user` | yes | yes | yes | yes | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Administrators can access all User Settings. | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |
| res.users.settings: access their own entries | `[Command.link(ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.res_users_settings_view_tree` | list |  | `id`, `user_id`, `use_push_to_talk` |  |  | `mail` |
| `mail.res_users_settings_view_form` | form |  | `user_id`, `is_discuss_sidebar_category_channel_open`, `is_discuss_sidebar_category_chat_open`, `use_push_to_talk`, `push_to_talk_key`, `voice_active_duration`, `volume_settings_ids`, `partner_id`, `volume` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.res_users_settings_action` | User Settings | list,form |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/res.users.settings.json`; views: `../../../schemas/interfaces/views/res.users.settings.json`.
