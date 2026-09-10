# Channel Member (`discuss.channel.member`)

**Transport name:** `discuss.channel.member`  
**Storage name:** `discuss_channel_member`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `im_livechat`

Description: Channel Member

## Identity and behavior

- Mixins (classical inheritance): `bus.listener.mixin`
- Display name search fields: `["channel_id", "partner_id", "guest_id"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (21)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Partner | many to one | `res.partner` | indexed; on delete of the target: cascade |
| `guest_id` | Guest | many to one | `mail.guest` | indexed; on delete of the target: cascade |
| `is_self` | Is Self | boolean |  | computed by rule `_compute_is_self` (not stored); searchable through a search rule |
| `channel_id` | Channel | many to one | `discuss.channel` | required; on delete of the target: cascade |
| `custom_channel_name` | Custom channel name | single line text |  |  |
| `fetched_message_id` | Last Fetched | many to one | `mail.message` | indexed (btree_not_null) |
| `seen_message_id` | Last Seen | many to one | `mail.message` | indexed (btree_not_null) |
| `new_message_separator` | New Message Separator | integer |  | required; default ; Help: Message id before which the separator should be displayed |
| `message_unread_counter` | Unread Messages Counter | integer |  | computed by rule `_compute_message_unread` (not stored) |
| `custom_notifications` | Customized Notifications | selection |  | Help: Use default from user settings if not specified. This setting will only be applied to channels. |
| `mute_until_dt` | Mute notifications until | date and time |  | Help: If set, the member will not receive notifications from the channel until this date. |
| `is_pinned` | Is pinned on the interface | boolean |  | computed by rule `_compute_is_pinned` (not stored); searchable through a search rule |
| `unpin_dt` | Unpin date | date and time |  | indexed; Help: Contains the date and time when the channel was unpinned by the user. |
| `last_interest_dt` | Last Interest | date and time |  | default computed dynamically (lambda self: fields.Datetime.now() - timedelta(seconds=1)); indexed; Help: Contains the date and time of the last interesting event that happened in this channel for this user. This includes: creating, joining, pinning |
| `last_seen_dt` | Last seen date | date and time |  |  |
| `rtc_session_ids` | RTC Sessions | one to many | `discuss.channel.rtc.session` | inverse field `channel_member_id` |
| `rtc_inviting_session_id` | Ringing session | many to one | `discuss.channel.rtc.session` |  |
| `livechat_member_history_ids` | Livechat Member History | one to many | `im_livechat.channel.member.history` | inverse field `member_id` |
| `livechat_member_type` | Livechat Member Type | selection |  | computed by rule `_compute_livechat_member_type` (not stored); writable through an inverse rule |
| `chatbot_script_id` | Chatbot Script | many to one | `chatbot.script` | computed by rule `_compute_chatbot_script_id` (not stored); writable through an inverse rule |
| `agent_expertise_ids` | Agent Expertise | many to many | `im_livechat.expertise` | computed by rule `_compute_agent_expertise_ids` (not stored); writable through an inverse rule |

## Selection values

### `custom_notifications` (Customized Notifications)

| Value | Label |
|---|---|
| `all` | All Messages |
| `mentions` | Mentions Only |
| `no_notif` | Nothing |

### `livechat_member_type` (Livechat Member Type)

| Value | Label |
|---|---|
| `agent` | Agent |
| `visitor` | Visitor |
| `bot` | Chatbot |

## Database constraints and indexes (4)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_seen_message_id_idx` | Index | `(channel_id, partner_id, seen_message_id)` |  | `mail` |
| `_partner_unique` | UniqueIndex | `(channel_id, partner_id) WHERE partner_id IS NOT NULL` |  | `mail` |
| `_guest_unique` | UniqueIndex | `(channel_id, guest_id) WHERE guest_id IS NOT NULL` |  | `mail` |
| `_partner_or_guest_exists` | Constraint | `CHECK((partner_id IS NOT NULL AND guest_id IS NULL) OR (partner_id IS NULL AND guest_id IS NOT NULL))` | A channel member must be a partner or a guest. | `mail` |

## Operations (41)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_gc_unpin_outdated_sub_channels` | background operation | self | `mail` | autovacuum |  |
| `_contrains_no_public_member` | validation | self | `mail` | constrains: `partner_id` |  |
| `_compute_is_self` | computation | self | `mail` | depends_context: `uid`, `guest` |  |
| `_search_is_self` | search rule | self, operator, operand | `mail` |  |  |
| `_search_is_pinned` | search rule | self, operator, operand | `mail` |  |  |
| `_compute_message_unread` | computation | self | `mail` | depends: `channel_id.message_ids`, `new_message_separator` |  |
| `_compute_display_name` | computation | self | `mail` | depends: `partner_id.name`, `guest_id.name`, `channel_id.display_name` |  |
| `_compute_is_pinned` | computation | self | `mail` | depends: `last_interest_dt`, `unpin_dt`, `channel_id.last_interest_dt` |  |
| `create` | lifecycle override | self, vals_list | `im_livechat`, `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `_sync_field_names` | internal rule | self | `mail` | model |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `_bus_channel` | internal rule | self | `mail` |  |  |
| `_notify_typing` | internal rule | self, is_typing | `mail` |  | Broadcast the typing notification to channel members :param is_typing: (boolean) tells whether the members are typing or not |
| `_notify_mute` | internal rule | self | `mail` |  |  |
| `_cleanup_expired_mutes` | internal rule | self | `mail` | model | Cron job for cleanup expired unmute by resetting mute_until_dt and sending bus notifications. |
| `_to_store_persona` | internal rule | self, fields | `mail` |  |  |
| `_to_store_defaults` | internal rule | self, target | `im_livechat`, `mail` |  |  |
| `_get_store_partner_fields` | preparation rule | self, fields | `im_livechat`, `mail` |  |  |
| `_get_store_guest_fields` | preparation rule | self, fields | `im_livechat`, `mail` |  |  |
| `_rtc_join_call` | internal rule | self, store, check_rtc_session_ids, camera | `mail` |  |  |
| `_join_sfu` | internal rule | self, ice_servers, force | `mail` |  |  |
| `_get_rtc_server_info` | preparation rule | self, rtc_session, ice_servers, key | `mail` |  |  |
| `_rtc_leave_call` | internal rule | self, session_id | `mail` |  |  |
| `_rtc_sync_sessions` | internal rule | self, check_rtc_session_ids | `mail` |  | Synchronize the RTC sessions for self channel member. - Inactive sessions of the channel are deleted. - Current sessions are returned. - Sessions given in check_rtc_session_ids that no longer exists   are returned as non-existing.  :param list check_rtc_session_ids: list of the ids of the sessions to check :returns: (current_rtc_sessions, outdated_rtc_sessions) :rtype: tuple |
| `_get_rtc_invite_members_domain` | preparation rule | self, member_ids | `im_livechat`, `mail` |  | Get the domain used to get the members to invite to and RTC call on the member's channel.  :param list member_ids: List of the partner ids to invite. |
| `_rtc_invite_members` | internal rule | self, member_ids | `mail` |  | Sends invitations to join the RTC call to all connected members of the thread who are not already invited, if member_ids is set, only the specified ids will be invited.  :param list member_ids: list of the partner ids to invite |
| `_mark_as_read` | internal rule | self, last_message_id | `mail` |  | Mark channel as read by updating the seen message id of the current member as well as its new message separator.  :param last_message_id: the id of the message to be marked as read. |
| `_set_last_seen_message` | internal rule | self, message, notify | `mail` |  | Set the last seen message of the current member.  :param message: the message to set as last seen message. :param notify: whether to send a bus notification relative to the new     last seen message. |
| `_set_new_message_separator` | internal rule | self, message_id | `mail` |  | :param message_id: id of the message above which the new message     separator should be displayed. |
| `_get_html_link_title` | preparation rule | self | `im_livechat`, `mail` |  |  |
| `_get_html_link` | preparation rule | self, *args, for_persona, **kwargs | `mail` |  |  |
| `_compute_livechat_member_type` | computation | self | `im_livechat` | depends: `livechat_member_history_ids.livechat_member_type` |  |
| `_compute_chatbot_script_id` | computation | self | `im_livechat` | depends: `livechat_member_history_ids.chatbot_script_id` |  |
| `_compute_agent_expertise_ids` | computation | self | `im_livechat` | depends: `livechat_member_history_ids.agent_expertise_ids` |  |
| `_create_or_update_history` | internal rule | self, values_by_member | `im_livechat` |  |  |
| `_inverse_livechat_member_type` | inverse computation | self | `im_livechat` |  |  |
| `_inverse_chatbot_script_id` | inverse computation | self | `im_livechat` |  |  |
| `_inverse_agent_expertise_ids` | inverse computation | self | `im_livechat` |  |  |
| `_gc_unpin_livechat_sessions` | background operation | self | `im_livechat` | autovacuum | Unpin read livechat sessions with no activity for at least one day to clean the operator's interface |
| `_get_excluded_rtc_members_partner_ids` | preparation rule | self | `im_livechat` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_contrains_no_public_member` | ValidationError | Channel members cannot include public users. | `mail` |
| `create` | UserError | It appears you're trying to create a channel member, but it seems like you forgot to specify the related channel. To move forward, please make sure to provide the necessary channel information. | `mail` |
| `create` | UserError | Adding more members to this chat isn't possible; it's designed for just two people. | `mail` |
| `write` | AccessError | You can not write on %(field_name)s. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | yes | yes | yes | yes | `mail` |
| `base.group_portal` | yes | yes | yes | yes | `mail` |
| `base.group_user` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| discuss.channel.member: sales users can read/create members on lead's origin channel | `[(4, ref('sales_team.group_sale_salesman'))]` | `[("channel_id.has_crm_lead", "=", True)]` | True | False | True | False |
| discuss.channel.member: livechat users can read all livechat channel members and can invite anyone | `[(4, ref('im_livechat_group_user'))]` | `[('channel_id.channel_type', '=', 'livechat')]` | True | False | True | False |
| discuss.channel.member: access their own entries | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     ('is_self', '=', True),                     "\|",                         ("channel_id.channel_type", "!=", "channel"),                         "\|",                             ("channel_id.group_public_id", "=", False),                             ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | False | True | False | True |
| discuss.channel.member: read members of accessible channels | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | True | False | False | False |
| discuss.channel.member: can join group restricted channels when group is matching | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     ('is_self', '=', True),                     ('channel_id.channel_type', '=', 'channel'),                     '\|',                         ('channel_id.group_public_id', '=', False),                         ('channel_id.group_public_id', 'in', user.all_group_ids.ids)                 ]` | False | False | True | False |
| discuss.channel.member: internal users can invite others in group restricted channels when group is matching | `[Command.link(ref('base.group_user'))]` | `[                     ('is_self', '=', False),                     ('channel_id.channel_type', '=', 'channel'),                     '\|',                         ('channel_id.group_public_id', '=', False),                         ('channel_id.group_public_id', 'in', user.all_group_ids.ids)                 ]` | False | False | True | False |
| discuss.channel.member: internal users can invite others in channels they are member of | `[Command.link(ref('base.group_user'))]` | `[                     ('is_self', '=', False),                     ('channel_id.channel_type', 'not in', ('channel', 'chat')),                     ('channel_id.is_member', '=', True)                 ]` | False | False | True | False |
| discuss.channel.member: admin can manipulate all entries | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.discuss_channel_member_view_tree` | list |  | `channel_id`, `partner_id`, `guest_id`, `is_pinned`, `last_seen_dt`, `last_interest_dt` |  |  | `mail` |
| `mail.discuss_channel_member_view_form` | form |  | `channel_id`, `partner_id`, `guest_id`, `custom_channel_name`, `fetched_message_id`, `seen_message_id`, `new_message_separator`, `message_unread_counter`, `custom_notifications`, `mute_until_dt`, `is_pinned`, `last_interest_dt`, `last_seen_dt`, `rtc_inviting_session_id` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.discuss_channel_member_action` | Channels/Members | list,form |  |  |  | `mail` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mail.discuss_channel_member_menu` | Channels/Members | `base.menu_email` | `mail.discuss_channel_member_action` | 21 | `base.group_no_one` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mail.ir_cron_discuss_channel_member_unmute` | Discuss: channel member unmute | 1 days | `_cleanup_expired_mutes` |  |

Machine-readable definition: `../../../schemas/data/entities/discuss.channel.member.json`; views: `../../../schemas/interfaces/views/discuss.channel.member.json`.
