# Discussion Channel (`discuss.channel`)

**Transport name:** `discuss.channel`  
**Storage name:** `discuss_channel`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `calendar`, `im_livechat`, `crm_livechat`, `hr`, `mail_bot`, `website_livechat`, `website_crm_livechat`

Description: Discussion Channel

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `bus.listener.mixin`, `rating.mixin`
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (66)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `active` | Active | boolean |  | default `True`; Help: Set active to false to hide the channel without removing it. |
| `channel_type` | Channel Type | selection |  | required; read only; default `channel`; on delete of the target: {"livechat": "cascade"}; Help: Chat is private and unique between 2 persons. Group is private among invited persons. Channel can be freely joined (depending on its configuration).; extended by packages `im_livechat` |
| `is_editable` | Is Editable | boolean |  | computed by rule `_compute_is_editable` (not stored) |
| `default_display_mode` | Default Display Mode | selection |  | Help: Determines how the channel will be displayed by default when opening it from its invitation link. No value means display text (no voice/video). |
| `description` | Description | multi line text |  |  |
| `image_128` | Image | image |  |  |
| `avatar_128` | Avatar | image |  | computed by rule `_compute_avatar_128` (not stored) |
| `avatar_cache_key` | Avatar Cache Key | single line text |  | computed by rule `_compute_avatar_cache_key` (not stored) |
| `channel_partner_ids` | Partners | many to many | `res.partner` | computed by rule `_compute_channel_partner_ids` (not stored); writable through an inverse rule; searchable through a search rule |
| `channel_member_ids` | Members | one to many | `discuss.channel.member` | inverse field `channel_id` |
| `parent_channel_id` | Parent Channel | many to one | `discuss.channel` | read only; indexed; on delete of the target: cascade; Help: Parent channel |
| `sub_channel_ids` | Sub Channels | one to many | `discuss.channel` | read only; inverse field `parent_channel_id` |
| `from_message_id` | From Message | many to one | `mail.message` | read only; Help: The message the channel was created from. |
| `pinned_message_ids` | Pinned Messages | one to many | `mail.message` | restricted by domain `[["model", "=", "discuss.channel"], ["pinned_at", "!=", false]]`; inverse field `res_id` |
| `sfu_channel_uuid` | Sfu Channel Uuid | single line text |  | visible only to groups `base.group_system` |
| `sfu_server_url` | Sfu Server Uniform resource locator | single line text |  | visible only to groups `base.group_system` |
| `rtc_session_ids` | Rtc Session | one to many | `discuss.channel.rtc.session` | visible only to groups `base.group_system`; inverse field `channel_id` |
| `call_history_ids` | Call History | one to many | `discuss.call.history` | inverse field `channel_id` |
| `is_member` | Is Member | boolean |  | computed by rule `_compute_is_member` (not stored); searchable through a search rule |
| `self_member_id` | Self Member | many to one | `discuss.channel.member` | computed by rule `_compute_self_member_id` (not stored) |
| `invited_member_ids` | Invited Member | one to many | `discuss.channel.member` | computed by rule `_compute_invited_member_ids` (not stored) |
| `member_count` | Member Count | integer |  | computed by rule `_compute_member_count` (not stored) |
| `message_count` | # Messages | integer |  | read only; computed by rule `_compute_message_count` (not stored) |
| `last_interest_dt` | Last Interest | date and time |  | default computed dynamically (lambda self: fields.Datetime.now() - timedelta(seconds=1)); indexed; Help: Contains the date and time of the last interesting event that happened in this channel. This updates itself when new message posted. |
| `group_ids` | Auto Subscription | many to many | `res.groups` | Help: Members of those groups will automatically added as followers. Note that they will be able to manage their subscription manually if necessary. |
| `uuid` | UUID | single line text |  | default computed dynamically (_generate_random_token); not copied on duplication; maximum length 50 |
| `group_public_id` | Authorized Group | many to one | `res.groups` | computed by rule `_compute_group_public_id` and stored; recursive dependency |
| `invitation_url` | Invitation uniform resource locator | single line text |  | computed by rule `_compute_invitation_url` (not stored) |
| `channel_name_member_ids` | Channel Name Member | one to many | `discuss.channel.member` | computed by rule `_compute_channel_name_member_ids` (not stored); Help: Members from which the channel name is computed when the name field is empty. |
| `calendar_event_ids` | Calendar Event | one to many | `calendar.event` | inverse field `videocall_channel_id` |
| `duration` | Duration | float |  | computed by rule `_compute_duration` (not stored); Help: Duration of the session in hours |
| `livechat_lang_id` | Language | many to one | `res.lang` | Help: Lang of the visitor of the channel. |
| `livechat_end_dt` | Session end date | date and time |  | Help: Session is closed when either the visitor or the last agent leaves the conversation. |
| `livechat_channel_id` | Channel | many to one | `im_livechat.channel` | indexed (btree_not_null) |
| `livechat_operator_id` | Operator | many to one | `res.partner` | indexed (btree_not_null) |
| `livechat_channel_member_history_ids` | Livechat Channel Member History | one to many | `im_livechat.channel.member.history` | inverse field `channel_id` |
| `livechat_expertise_ids` | Livechat Expertise | many to many | `im_livechat.expertise` | association table `discuss_channel_im_livechat_expertise_rel` |
| `livechat_agent_history_ids` | Agents (History) | one to many | `im_livechat.channel.member.history` | computed by rule `_compute_livechat_agent_history_ids` (not stored); searchable through a search rule |
| `livechat_bot_history_ids` | Bots (History) | one to many | `im_livechat.channel.member.history` | computed by rule `_compute_livechat_bot_history_ids` (not stored); searchable through a search rule |
| `livechat_customer_history_ids` | Customers (History) | one to many | `im_livechat.channel.member.history` | computed by rule `_compute_livechat_customer_history_ids` (not stored); searchable through a search rule |
| `livechat_agent_partner_ids` | Agents | many to many | `res.partner` | computed by rule `_compute_livechat_agent_partner_ids` and stored; association table `im_livechat_channel_member_history_discuss_channel_agent_rel` |
| `livechat_bot_partner_ids` | Bots | many to many | `res.partner` | computed by rule `_compute_livechat_bot_partner_ids` and stored; association table `im_livechat_channel_member_history_discuss_channel_bot_rel` |
| `livechat_customer_partner_ids` | Customers (Partners) | many to many | `res.partner` | computed by rule `_compute_livechat_customer_partner_ids` and stored; association table `im_livechat_channel_member_history_discuss_channel_customer_rel` |
| `livechat_customer_guest_ids` | Customers (Guests) | many to many | `mail.guest` | computed by rule `_compute_livechat_customer_guest_ids` (not stored) |
| `livechat_agent_requesting_help_history` | Help Requested (Agent) | many to one | `im_livechat.channel.member.history` | computed by rule `_compute_livechat_agent_requesting_help_history` and stored |
| `livechat_agent_providing_help_history` | Help Provided (Agent) | many to one | `im_livechat.channel.member.history` | computed by rule `_compute_livechat_agent_providing_help_history` and stored |
| `livechat_note` | Live Chat Note | rich text |  | visible only to groups `base.group_user`; Help: Note about the session, visible to all internal users having access to the session. |
| `livechat_status` | Livechat Status | selection |  | computed by rule `_compute_livechat_status` and stored; visible only to groups `base.group_user` |
| `livechat_outcome` | Livechat Outcome | selection |  | computed by rule `_compute_livechat_outcome` and stored |
| `livechat_conversation_tag_ids` | Live Chat Conversation Tags | many to many | `im_livechat.conversation.tag` | visible only to groups `im_livechat.im_livechat_group_user`; association table `livechat_conversation_tag_rel`; Help: Tags to qualify the conversation. |
| `livechat_start_hour` | Session Start Hour | float |  | computed by rule `_compute_livechat_start_hour` and stored |
| `livechat_week_day` | Day of the Week | selection |  | computed by rule `_compute_livechat_week_day` and stored |
| `livechat_matches_self_lang` | Livechat Matches Self Lang | boolean |  | computed by rule `_compute_livechat_matches_self_lang` (not stored); searchable through a search rule |
| `livechat_matches_self_expertise` | Livechat Matches Self Expertise | boolean |  | computed by rule `_compute_livechat_matches_self_expertise` (not stored); searchable through a search rule |
| `chatbot_current_step_id` | Chatbot Current Step | many to one | `chatbot.script.step` |  |
| `chatbot_message_ids` | Chatbot Messages | one to many | `chatbot.message` | visible only to groups `im_livechat.im_livechat_group_manager`; inverse field `discuss_channel_id` |
| `country_id` | Country | many to one | `res.country` | Help: Country of the visitor of the channel |
| `livechat_failure` | Live Chat Session Failure | selection |  |  |
| `livechat_is_escalated` | Is session escalated | boolean |  | computed by rule `_compute_livechat_is_escalated` and stored |
| `rating_last_text` | Rating Last Text | selection |  |  |
| `lead_ids` | Leads | one to many | `crm.lead` | visible only to groups `sales_team.group_sale_salesman`; inverse field `origin_channel_id`; Help: The channel becomes accessible to sales users when leads are set. |
| `has_crm_lead` | Has Customer relationship management Lead | boolean |  | computed by rule `_compute_has_crm_lead` and stored |
| `subscription_department_ids` | human resources Departments | many to many | `hr.department` | Help: Automatically subscribe members of those departments to the channel. |
| `is_pending_chat_request` | When created from an operator, whether the channel is yet to be opened on the visitor side. | boolean |  |  |
| `livechat_visitor_id` | Visitor | many to one | `website.visitor` | indexed (btree_not_null) |

## Selection values

### `channel_type` (Channel Type)

| Value | Label |
|---|---|
| `chat` | Chat |
| `channel` | Channel |
| `group` | Group |
| `livechat` | Livechat Conversation |

### `default_display_mode` (Default Display Mode)

| Value | Label |
|---|---|
| `video_full_screen` | Full screen video |

### `livechat_status` (Livechat Status)

| Value | Label |
|---|---|
| `in_progress` | In progress |
| `waiting` | Waiting for customer |
| `need_help` | Looking for help |

### `livechat_outcome` (Livechat Outcome)

| Value | Label |
|---|---|
| `no_answer` | Never Answered |
| `no_agent` | No one Available |
| `no_failure` | Success |
| `escalated` | Escalated |

### `livechat_week_day` (Day of the Week)

| Value | Label |
|---|---|
| `0` | Monday |
| `1` | Tuesday |
| `2` | Wednesday |
| `3` | Thursday |
| `4` | Friday |
| `5` | Saturday |
| `6` | Sunday |

### `livechat_failure` (Live Chat Session Failure)

| Value | Label |
|---|---|
| `no_answer` | Never Answered |
| `no_agent` | No one Available |
| `no_failure` | No Failure |

## State fields

State machine fields of this entity: `livechat_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (10)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_from_message_id_unique` | Constraint | `UNIQUE(from_message_id)` | Messages can only be linked to one sub-channel | `mail` |
| `_uuid_unique` | Constraint | `UNIQUE(uuid)` | The channel UUID must be unique | `mail` |
| `_group_public_id_check` | Constraint | `CHECK (channel_type = 'channel' OR group_public_id IS NULL)` | Group authorization and group auto-subscription are only supported on channels. | `mail` |
| `_livechat_operator_id` | Constraint | `CHECK((channel_type = 'livechat' and livechat_operator_id is not null) or (channel_type != 'livechat'))` | Livechat Operator ID is required for a channel of type livechat. | `im_livechat` |
| `_livechat_end_dt_status_constraint` | Constraint | `CHECK(livechat_end_dt IS NULL or livechat_status IS NULL)` | Closed Live Chat session should not have a status. | `im_livechat` |
| `_livechat_end_dt_idx` | Index | `(livechat_end_dt) WHERE livechat_end_dt IS NULL` |  | `im_livechat` |
| `_livechat_failure_idx` | Index | `(livechat_failure) WHERE livechat_failure IN ('no_answer', 'no_agent')` |  | `im_livechat` |
| `_livechat_is_escalated_idx` | Index | `(livechat_is_escalated) WHERE livechat_is_escalated IS TRUE` |  | `im_livechat` |
| `_livechat_channel_type_create_date_idx` | Index | `(channel_type, create_date) WHERE channel_type = 'livechat'` |  | `im_livechat` |
| `_has_crm_lead_index` | Index | `(has_crm_lead) WHERE has_crm_lead IS TRUE` |  | `crm_livechat` |

## Operations (132)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_generate_random_token` | internal rule | self | `mail` | model |  |
| `_constraint_from_message_id` | validation | self | `mail` | constrains: `from_message_id` |  |
| `_constraint_parent_channel_id` | validation | self | `mail` | constrains: `parent_channel_id` |  |
| `_constraint_partners_chat` | validation | self | `mail` | constrains: `channel_member_ids` |  |
| `_constraint_group_id_channel` | validation | self | `mail` | constrains: `group_public_id`, `group_ids` |  |
| `_compute_display_name` | computation | self | `mail` | depends: `channel_name_member_ids`, `name` |  |
| `_compute_channel_name_member_ids` | computation | self | `mail` | depends: `channel_member_ids` |  |
| `_compute_is_editable` | computation | self | `mail` | depends: `channel_type`, `is_member`, `group_public_id`; depends_context: `uid` |  |
| `_compute_avatar_128` | computation | self | `mail` | depends: `channel_type`, `image_128`, `uuid` |  |
| `_compute_avatar_cache_key` | computation | self | `mail` | depends: `avatar_128` |  |
| `_generate_avatar` | internal rule | self | `mail` |  |  |
| `_compute_channel_partner_ids` | computation | self | `mail` | depends: `channel_member_ids.partner_id` |  |
| `_inverse_channel_partner_ids` | inverse computation | self | `mail` |  |  |
| `_search_channel_partner_ids` | search rule | self, operator, operand | `mail` |  |  |
| `_compute_is_member` | computation | self | `mail` | depends_context: `uid`, `guest`; depends: `channel_member_ids` |  |
| `_search_is_member` | search rule | self, operator, operand | `mail` |  |  |
| `_compute_self_member_id` | computation | self | `mail` | depends_context: `uid`, `guest`; depends: `channel_member_ids` |  |
| `_compute_invited_member_ids` | computation | self | `mail` | depends: `channel_member_ids.rtc_inviting_session_id` |  |
| `_compute_member_count` | computation | self | `mail` | depends: `channel_member_ids` |  |
| `_compute_message_count` | computation | self | `mail` | depends: `message_ids` |  |
| `_compute_group_public_id` | computation | self | `mail` | depends: `channel_type`, `parent_channel_id.group_public_id` |  |
| `_compute_invitation_url` | computation | self | `mail` | depends: `uuid` |  |
| `_get_allowed_channel_member_create_params` | preparation rule | self | `im_livechat`, `mail` | model |  |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `_unlink_except_all_employee_channel` | internal rule | self | `mail` | ondelete |  |
| `write` | lifecycle override | self, vals | `hr`, `im_livechat`, `mail` |  |  |
| `_sync_field_names` | internal rule | self | `im_livechat`, `mail` |  |  |
| `_subscribe_users_automatically` | internal rule | self | `mail` |  |  |
| `_subscribe_users_automatically_get_members` | internal rule | self | `hr`, `mail` |  | Return new members per channel ID |
| `action_unfollow` | user action | self | `mail` |  |  |
| `_action_unfollow` | internal rule | self, partner, guest, post_leave_message | `im_livechat`, `mail` |  |  |
| `add_members` | operation | self, partner_ids, guest_ids, invite_to_rtc_call, post_joined_message | `mail` |  | Adds the given partner_ids and guest_ids as member of self channels. |
| `_add_members` | internal rule | self, guests, partners, users, create_member_params, invite_to_rtc_call, post_joined_message, inviting_partner | `im_livechat`, `mail` |  |  |
| `invite_by_email` | operation | self, emails | `mail` |  | Send channel invitation emails to a list of email addresses. Existing members' email addresses are ignored.  :param emails: List of email addresses to invite. :type emails: list[str] |
| `_get_call_notification_tag` | preparation rule | self | `mail` |  |  |
| `_rtc_cancel_invitations` | internal rule | self, member_ids | `mail` |  | Cancels the invitations of the RTC call from all invited members, if member_ids is provided, only the invitations of the specified members are canceled.  :param list member_ids: list of the members ids from which the invitation has to be removed |
| `_notify_get_recipients` | internal rule | self, message, msg_vals, **kwargs | `mail` |  |  |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `mail` |  |  |
| `_get_notify_valid_parameters` | preparation rule | self | `mail` |  |  |
| `_notify_thread` | internal rule | self, message, msg_vals, **kwargs | `mail` |  |  |
| `_notify_by_web_push_prepare_payload` | internal rule | self, message, msg_vals, force_record_name | `mail` |  |  |
| `_notify_thread_by_web_push` | internal rule | self, message, recipients_data, msg_vals, **kwargs | `mail` |  |  |
| `_message_receive_bounce` | messaging hook | self, email, partner | `mail` |  |  |
| `_get_allowed_message_params` | preparation rule | self | `mail` |  |  |
| `_get_allowed_message_partner_ids` | preparation rule | self, partner_ids | `mail` |  | Ensure only partners having access to the channel can be mentioned. |
| `message_post` | messaging hook | self, message_type, partner_ids, **kwargs | `mail`, `website_livechat` |  | Override to mark the visitor as still connected. If the message sent is not from the operator (so if it's the visitor or system_robot sending closing chat notification, the visitor last action date is updated. |
| `_message_post_after_hook` | messaging hook | self, message, msg_vals | `im_livechat`, `mail_bot`, `mail` |  | This method is called just before _notify_thread() method which is calling the _to_store() method. We need a 'chatbot.message' record before it happens to correctly display the message. It's created only if the mail channel is linked to a chatbot step. We also need to save the user answer if the current step is a question selection. |
| `_message_update_content` | messaging hook | partner_ids, **kwargs | `mail` |  |  |
| `_check_can_update_message_content` | validation | self, message | `mail` |  |  |
| `_create_attachments_for_post` | internal rule | self, values_list, extra_list | `mail` |  |  |
| `_message_subscribe` | messaging hook | self, partner_ids, subtype_ids, customer_ids | `mail` |  |  |
| `_should_invite_members_to_join_call` | internal rule | self | `calendar`, `mail` |  |  |
| `_get_access_action` | preparation rule | self, access_uid, force_website | `mail` |  | Redirect to Discuss instead of form view. |
| `_broadcast` | internal rule | self, partner_ids | `mail` |  | Broadcast the current channel header to the given partner ids :param partner_ids : the partner to notify |
| `set_message_pin` | operation | self, message_id, pinned | `mail` |  | (Un)pin a message on the channel and send a notification to the members. :param message_id: id of the message to be pinned. :param pinned: whether the message should be pinned or unpinned. |
| `_find_or_create_member_for_self` | internal rule | self | `mail` |  |  |
| `_find_or_create_persona_for_channel` | internal rule | self, guest_name, timezone, country_code, create_member_params, post_joined_message | `mail` |  | :param guest_name: name of the persona :param post_joined_message: whether to post a message to the channel     to notify that the persona joined  :param dict create_member_params: optional parameters to pass to the     channel member create function.  :rtype: tuple[partner, guest] |
| `_get_channels_as_member` | preparation rule | self | `mail` | model |  |
| `_to_store_defaults` | internal rule | self, target | `im_livechat`, `mail`, `website_livechat` |  |  |
| `_to_store` | internal rule | self, store, fields | `im_livechat`, `mail` |  | Extends the channel header by adding the livechat operator and the 'anonymous' profile |
| `_get_or_create_chat` | preparation rule | self, partners_to, pin | `mail` | model | Get the canonical private channel between some partners, create it if needed. To reuse an old channel (conversation), this one must be private, and contains only the given partners. :param partners_to : list of res.partner ids to add to the conversation :param pin : True if getting the channel should pin it for the current user :returns: channel_info of the created or existing channel :rtype: dict |
| `channel_pin` | operation | self, pinned | `mail`, `website_livechat` |  | Override to clean an empty livechat channel. This is typically called when the operator send a chat request to a website.visitor but don't speak to them and closes the chatter. This allows operators to send the visitor a new chat request. If active empty livechat channel, delete discuss_channel as not useful to keep empty chat |
| `_allow_invite_by_email` | internal rule | self | `mail` |  |  |
| `_types_allowing_seen_infos` | internal rule | self | `im_livechat`, `mail` |  | Return the channel types which allow sending seen infos notification on the channel |
| `_types_allowing_unfollow` | internal rule | self | `im_livechat`, `mail` |  | Return the channel types which allow leaving the channel, channel will be unpinned otherwise |
| `_member_based_naming_channel_types` | internal rule | self | `mail` |  | Return the channel types that use member-based naming, specifically the `channel_name_member_ids` field. |
| `_lazy_load_members_channel_types` | internal rule | self | `mail` |  | Return the channel types that load members lazily. |
| `channel_fetched` | operation | self | `mail` |  | Broadcast the channel_fetched notification to channel members |
| `channel_set_custom_name` | operation | self, name | `mail` |  |  |
| `channel_rename` | operation | self, name | `mail` |  |  |
| `channel_change_description` | operation | self, description | `mail` |  |  |
| `channel_join` | operation | self | `mail` |  | Shortcut to add the current user as member of self channels. Prefer calling add_members() directly when possible. |
| `_create_channel` | internal rule | self, name, group_id | `mail` | model | Create a channel and add the current partner, broadcast it (to make the user directly listen to it when polling) :param name : the name of the channel to create :param group_id : the group allowed to join the channel. :return dict : channel header |
| `_create_group` | internal rule | self, partners_to, default_display_mode, name | `mail` | model | Creates a group channel.  :param partners_to : list of res.partner ids to add to the conversation :param str default_display_mode: how the channel will be displayed by default :param str name: group name. default name is computed client side from the list of members if no name is set :returns: channel_info of the created channel :rtype: dict |
| `_create_sub_channel` | internal rule | self, from_message_id, name | `mail` |  |  |
| `get_mention_suggestions` | operation | self, search, limit | `mail` | readonly; model | Return 'limit'-first channels' name, channel_type and group_public_id fields such that the name matches a 'search' string. Exclude channels of type chat (DM) and group. |
| `_get_last_messages` | preparation rule | self | `mail` |  | Return the last message for each of the given channels. |
| `_clean_empty_message` | internal rule | self, message | `mail` |  |  |
| `_get_store_message_update_extra_fields` | preparation rule | self | `mail` |  |  |
| `execute_command_help` | operation | self, **kwargs | `mail_bot`, `mail` |  |  |
| `_execute_command_help_message_extra` | internal rule | self | `mail` |  |  |
| `execute_command_leave` | operation | self, **kwargs | `mail` |  |  |
| `execute_command_who` | operation | self, **kwargs | `mail` |  |  |
| `_compute_duration` | computation | self | `im_livechat` | depends: `livechat_end_dt` |  |
| `_compute_livechat_status` | computation | self | `im_livechat` | depends: `livechat_end_dt` |  |
| `_compute_livechat_is_escalated` | computation | self | `im_livechat` | depends: `livechat_agent_history_ids` |  |
| `_compute_livechat_agent_history_ids` | computation | self | `im_livechat` | depends: `livechat_channel_member_history_ids.livechat_member_type` |  |
| `_compute_livechat_bot_history_ids` | computation | self | `im_livechat` | depends: `livechat_channel_member_history_ids.livechat_member_type` |  |
| `_search_livechat_bot_history_ids` | search rule | self, operator, value | `im_livechat` |  |  |
| `_compute_livechat_customer_history_ids` | computation | self | `im_livechat` | depends: `livechat_channel_member_history_ids.livechat_member_type` |  |
| `_search_livechat_customer_history_ids` | search rule | self, operator, value | `im_livechat` |  |  |
| `_compute_livechat_agent_partner_ids` | computation | self | `im_livechat` | depends: `livechat_agent_history_ids.partner_id` |  |
| `_search_livechat_agent_history_ids` | search rule | self, operator, value | `im_livechat` |  |  |
| `_compute_livechat_bot_partner_ids` | computation | self | `im_livechat` | depends: `livechat_bot_history_ids.partner_id` |  |
| `_compute_livechat_customer_partner_ids` | computation | self | `im_livechat` | depends: `livechat_customer_history_ids.partner_id` |  |
| `_compute_livechat_customer_guest_ids` | computation | self | `im_livechat` |  |  |
| `_compute_livechat_agent_requesting_help_history` | computation | self | `im_livechat` | depends: `livechat_agent_history_ids` |  |
| `_compute_livechat_agent_providing_help_history` | computation | self | `im_livechat` | depends: `livechat_agent_history_ids` |  |
| `_compute_livechat_outcome` | computation | self | `im_livechat` | depends: `livechat_is_escalated`, `livechat_failure` |  |
| `_compute_livechat_matches_self_lang` | computation | self | `im_livechat` | depends_context: `user` |  |
| `_search_livechat_matches_self_lang` | search rule | self, operator, value | `im_livechat` |  |  |
| `_compute_livechat_matches_self_expertise` | computation | self | `im_livechat` | depends_context: `user` |  |
| `_search_livechat_matches_self_expertise` | search rule | self, operator, value | `im_livechat` |  |  |
| `_compute_livechat_start_hour` | computation | self | `im_livechat` | depends: `create_date` |  |
| `_compute_livechat_week_day` | computation | self | `im_livechat` | depends: `create_date` |  |
| `_store_livechat_operator_id_fields` | internal rule | self | `im_livechat` |  | Return the standard fields to include in Store for livechat_operator_id. |
| `_gc_empty_livechat_sessions` | background operation | self | `im_livechat` | autovacuum |  |
| `_gc_bot_only_ongoing_sessions` | background operation | self | `im_livechat` | autovacuum | Garbage collect bot-only livechat sessions with no activity for over 1 day. |
| `execute_command_history` | operation | self, **kwargs | `im_livechat` |  |  |
| `_get_visitor_leave_message` | preparation rule | self, operator, cancel | `im_livechat`, `website_livechat` |  |  |
| `_close_livechat_session` | internal rule | self, **kwargs | `im_livechat` |  | Set deactivate the livechat channel and notify (the operator) the reason of closing the session. |
| `_rating_get_parent_field_name` | internal rule | self | `im_livechat` |  |  |
| `_email_livechat_transcript` | internal rule | self, email | `im_livechat` |  |  |
| `_attachment_to_html` | internal rule | self, attachment | `im_livechat` |  |  |
| `_get_channel_history` | preparation rule | self | `im_livechat` |  | Converting message body back to plaintext for correct data formatting in HTML field. |
| `_get_livechat_session_fields_to_store` | preparation rule | self | `crm_livechat`, `im_livechat`, `website_livechat` |  |  |
| `_chatbot_find_customer_values_in_messages` | internal rule | self, step_type_to_field | `im_livechat` |  | Look for user's input in the channel's messages based on a dictionary mapping the step_type to the field name of the model it will be used on.  :param dict step_type_to_field: a dict of step types to customer fields     to fill, like : {'question_email': 'email_from', 'question_phone': 'mobile'} |
| `_chatbot_post_message` | internal rule | self, chatbot_script, body | `im_livechat` |  | Small helper to post a message as the chatbot operator  :param record chatbot_script :param string body: message HTML body |
| `_chatbot_validate_email` | internal rule | self, email_address, chatbot_script | `im_livechat` |  |  |
| `_chatbot_restart` | internal rule | self, chatbot_script | `im_livechat` |  |  |
| `livechat_join_channel_needing_help` | operation | self | `im_livechat` |  | Join a live chat for which help was requested.  :returns: Whether the live chat was joined. False if the live chat could not     be joined because another agent already joined the channel in the meantime. :rtype: bool |
| `_forward_human_operator` | internal rule | self, chatbot_script_step, users | `im_livechat` |  | Add a human operator to the conversation. The conversation with the chatbot (scripted chatbot or ai agent) is stopped the visitor will continue the conversation with a real person.  In case we don't find any operator (e.g: no-one is available) we don't post any messages. The chat with the chatbot will continue normally, which allows to add extra steps when it's the case (e.g: ask for the visitor's email and create a lead).  :param chatbot_script_step: the forward to operator chatbot script step if the forwarding is done through a scripted chatbot (not used if the forwarding is done through an  |
| `_get_human_operator` | preparation rule | self, users, chatbot_script_step | `im_livechat` |  |  |
| `_post_current_chatbot_step_message` | internal rule | self, chatbot_script_step | `im_livechat` |  |  |
| `_add_new_members_to_channel` | internal rule | self, create_member_params, inviting_partner, users, partners | `im_livechat` |  |  |
| `_update_forwarded_channel_data` | internal rule | livechat_failure, livechat_operator_id, operator_name | `im_livechat` |  |  |
| `_add_next_step_message_to_store` | internal rule | self, chatbot_script_step | `im_livechat` |  |  |
| `_compute_has_crm_lead` | computation | self | `crm_livechat` | depends: `lead_ids` |  |
| `execute_command_lead` | operation | self, **kwargs | `crm_livechat` |  |  |
| `_convert_visitor_to_lead` | internal rule | self, partner, key | `crm_livechat`, `website_crm_livechat` |  | Create a lead from channel /lead command :param partner: internal user partner (operator) that created the lead; :param key: operator input in chat ('/lead Lead about Product') |
| `_constraint_subscription_department_ids_channel` | validation | self | `hr` | constrains: `subscription_department_ids` |  |
| `_get_visitor_history` | preparation rule | self, visitor | `website_livechat` |  |  |

## Validation and error messages (18)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constraint_from_message_id` | ValidationError | Cannot create %(channels)s: initial message should belong to parent channel or one of its sub-channels. | `mail` |
| `_constraint_parent_channel_id` | ValidationError | Cannot create %(channels)s: parent should not be a sub-channel and should be of type 'channel' or 'group'. The sub-channel should have the same type as the parent. | `mail` |
| `_constraint_partners_chat` | ValidationError | A channel of type 'chat' cannot have more than two users. | `mail` |
| `_constraint_group_id_channel` | ValidationError | For %(channels)s, channel_type should be 'channel' to have the group-based authorization or group auto-subscription. | `mail` |
| `create` | ValidationError | Invalid value when creating a channel with members, only 4 or 6 are allowed. | `mail` |
| `create` | ValidationError | Invalid value when creating a channel with memberships, only 0 is allowed. | `mail` |
| `create` | ValidationError | Invalid field “%(field_name)s” when creating a channel with members. | `mail` |
| `_unlink_except_all_employee_channel` | UserError | You cannot delete those groups, as the Whole Company group is required by other modules. | `mail` |
| `write` | UserError | Cannot change initial message nor parent channel of: %(channels)s. | `mail` |
| `write` | UserError | Cannot change the channel type of: %(channel_names)s | `mail` |
| `write` | UserError | Cannot change authorized group of sub-channel: %(channels)s. | `mail` |
| `invite_by_email` | AccessError | You don't have access to invite users to this channel. | `mail` |
| `invite_by_email` | UserError | Inviting by email is not allowed for this channel type (%s). | `mail` |
| `invite_by_email` | UserError | error_msg | `mail` |
| `_check_can_update_message_content` | UserError | Only messages type comment can have their content updated on model 'discuss.channel' | `mail` |
| `_message_subscribe` | UserError | Adding followers on channels is not possible. Consider adding members instead. | `mail` |
| `_get_or_create_chat` | UserError | A chat should not be created with more than 2 persons. Create a group instead. | `mail` |
| `_constraint_subscription_department_ids_channel` | ValidationError | For %(channels)s, channel_type should be 'channel' to have the department auto-subscription. | `hr` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `mail` |
| `base.group_portal` | no | yes | no | no | `mail` |
| `base.group_user` | yes | yes | yes | no | `mail` |
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| discuss.channel: sales users can read lead's origin channel | `[(4, ref('sales_team.group_sale_salesman'))]` | `[("has_crm_lead", "=", True)]` | True | False | False | False |
| discuss.channel: livechat users can read all livechat channels | `[(4, ref('im_livechat_group_user'))]` | `[('channel_type', '=', 'livechat')]` | True | False | False | False |
| discuss.channel: can access channels (as member or as group allowed) | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     "\|",                         "&",                             ("channel_type", "!=", "channel"),                             "\|",                                 ("is_member", "=", True),                                 ("parent_channel_id.is_member", "=", True),                         "&",                             ("channel_type", "=", "channel"),                             "\|",                                 ("group_public_id", "=", False),                                 ("group_public_id", "in", user.all_group_ids.ids),                 ]` | True | True | True | True |
| discuss.channel: admin full access | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (18)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm_livechat.discuss_channel_view_form` | xpath | `mail.discuss_channel_view_form` | `has_crm_lead`, `lead_ids` |  |  | `crm_livechat` |
| `hr.discuss_channel_view_form` | xpath | `mail.discuss_channel_view_form` | `subscription_department_ids` |  |  | `hr` |
| `hr_livechat.discuss_channel_view_search` | xpath | `im_livechat.discuss_channel_view_search` |  |  | `My Team` | `hr_livechat` |
| `hr_livechat.discuss_channel_looking_for_help_view_search` | xpath | `im_livechat.discuss_channel_looking_for_help_view_search` |  |  | `My Team` | `hr_livechat` |
| `im_livechat.discuss_channel_view_search` | search |  | `livechat_agent_partner_ids`, `livechat_agent_requesting_help_history`, `livechat_agent_providing_help_history`, `country_id`, `livechat_customer_partner_ids` |  | `My Sessions`, `Ongoing`, `Happy`, `Neutral`, `Unhappy`, `Unrated`, `Session Date`, `Last 24 Hours`, `Last 7 Days`, `Last 30 Days`, `Last 365 Days`, `Escalated`, `Handled by Agent`, `Handled by Bot`, `In Call`, `Channel`, `Agent`, `group_by_agent_requesting_help`, `group_by_agent_providing_help`, `Rating`, `Country`, `Customer`, `Session Date` | `im_livechat` |
| `im_livechat.discuss_channel_view_tree` | list |  | `create_date`, `livechat_customer_history_ids`, `livechat_agent_history_ids`, `livechat_agent_requesting_help_history`, `livechat_agent_providing_help_history`, `livechat_bot_history_ids`, `country_id`, `livechat_lang_id`, `livechat_expertise_ids`, `livechat_conversation_tag_ids`, `livechat_channel_id`, `duration`, `message_count`, `rating_last_text`, `rating_last_feedback` |  |  | `im_livechat` |
| `im_livechat.discuss_channel_view_kanban` | kanban |  | `livechat_customer_history_ids`, `create_date`, `duration`, `message_count`, `livechat_failure`, `livechat_is_escalated`, `rating_last_image`, `country_id`, `livechat_agent_partner_ids` |  |  | `im_livechat` |
| `im_livechat.discuss_channel_view_form` | form |  | `rating_last_image`, `rating_last_feedback`, `name`, `create_date` |  |  | `im_livechat` |
| `im_livechat.discuss_channel_view_pivot` | pivot |  | `livechat_operator_id`, `create_date`, `rating_last_value` |  |  | `im_livechat` |
| `im_livechat.discuss_channel_view_graph` | graph |  | `create_date`, `rating_last_value` |  |  | `im_livechat` |
| `im_livechat.discuss_channel_looking_for_help_view_search` | search |  | `livechat_agent_partner_ids`, `description`, `country_id`, `livechat_lang_id`, `livechat_expertise_ids`, `livechat_conversation_tag_ids`, `livechat_bot_partner_ids`, `livechat_customer_partner_ids` |  | `My Sessions`, `My Languages`, `My Expertise`, `Session Date`, `Last 24 Hours`, `Last 7 Days`, `Last 30 Days`, `Last 365 Days`, `Channel`, `Agent`, `Country`, `Language`, `Expertise`, `Tags`, `Chatbot`, `Customer`, `Session Date` | `im_livechat` |
| `im_livechat.discuss_channel_looking_for_help_view_list` | list |  | `create_date`, `description`, `livechat_agent_history_ids`, `livechat_bot_history_ids`, `livechat_customer_history_ids`, `country_id`, `livechat_lang_id`, `livechat_expertise_ids`, `livechat_conversation_tag_ids`, `livechat_channel_id`, `duration`, `message_count` |  |  | `im_livechat` |
| `im_livechat.discuss_channel_looking_for_help_view_kanban` | kanban |  | `livechat_customer_history_ids`, `create_date`, `country_id`, `livechat_lang_id`, `livechat_expertise_ids`, `livechat_agent_partner_ids` |  |  | `im_livechat` |
| `mail.discuss_channel_view_kanban` | kanban |  | `is_member`, `group_ids`, `active`, `avatar_128`, `name`, `channel_type`, `description` | `channel_join`, `action_unfollow` |  | `mail` |
| `mail.discuss_channel_view_list` | list |  | `avatar_128`, `name`, `description` | `Join`, `Leave` |  | `mail` |
| `mail.discuss_channel_view_form` | form |  | `avatar_128`, `is_editable`, `image_128`, `name`, `active`, `description`, `group_public_id`, `group_ids`, `channel_type`, `channel_member_ids`, `partner_id`, `guest_id`, `channel_type`, `parent_channel_id`, `sub_channel_ids`, `from_message_id`, `sfu_channel_uuid`, `sfu_server_url` |  |  | `mail` |
| `mail.discuss_channel_view_tree` | list |  | `name` |  |  | `mail` |
| `mail.discuss_channel_view_search` | search |  | `name` |  | `Archived` | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `im_livechat.discuss_channel_action` | Sessions | kanban,list,pivot,graph,form | `[('livechat_channel_id', '!=', None)]` | `{                     "search_default_filter_session_date": "custom_rated_on_last_30_days",                     "im_livechat_hide_partner_company": True,                 }` |  | `im_livechat` |
| `im_livechat.discuss_channel_action_from_livechat_channel` | Sessions | kanban,list,pivot,graph,form | `[('livechat_channel_id', 'in', [active_id])]` | `{                 'search_default_livechat_channel_id': [active_id],                 'default_livechat_channel_id': active_id,             }` |  | `im_livechat` |
| `im_livechat.discuss_channel_looking_for_help_action` | Looking for Help | list,kanban,form | `[("livechat_status", "=", "need_help")]` | `{                     "search_default_filter_session_date": "custom_created_on_last_30_days",                     "im_livechat_hide_partner_company": True,                 }` |  | `im_livechat` |
| `mail.discuss_channel_action_view` | Join a group | kanban,list,form |  |  |  | `mail` |
| `mail.discuss_channel_action` | Channels | kanban,form | `[(('channel_type', '=', 'channel'))]` |  |  | `mail` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_all_action` | Sessions |  | `[('channel_type', '=', 'livechat')]` | `{'search_default_ongoing': 1}` |  | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_escalated_action` | Sessions |  |  | `{'search_default_ongoing': 1, 'search_default_escalated': 1}` |  | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_agents_in_call_action` | Sessions |  |  | `{'search_default_ongoing': 1, 'search_default_in_call': 1}` |  | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_handle_by_agent_action` | Sessions |  |  | `{'search_default_ongoing': 1, 'search_default_handled_by_agent': 1}` |  | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_handle_by_bot_action` | Sessions |  |  | `{'search_default_ongoing': 1, 'search_default_handled_by_bot': 1}` |  | `spreadsheet_dashboard_im_livechat` |
| `website_livechat.website_visitor_livechat_session_action` | Visitor's Sessions | list,form | `[('livechat_visitor_id', '=', active_id), ('has_message', '=', True)]` | `{                 'search_default_livechat_visitor_id': [active_id],                 'default_livechat_visitor_id': active_id,             }` |  | `website_livechat` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mail.menu_channel` | Channels | `mail.menu_root_discuss` | `mail.discuss_channel_action` | 2 |  |
| `mail.discuss_channel_menu_settings` | Channels | `base.menu_email` | `mail.discuss_channel_action_view` | 20 | `base.group_no_one` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `im_livechat.action_report_livechat_conversation` | Live Chat Conversation | qweb-pdf | `im_livechat.report_livechat_conversation` |  |  |

Machine-readable definition: `../../../schemas/data/entities/discuss.channel.json`; views: `../../../schemas/interfaces/views/discuss.channel.json`.
