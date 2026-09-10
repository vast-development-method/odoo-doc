# Livechat Channel (`im_livechat.channel`)

**Transport name:** `im_livechat.channel`  
**Storage name:** `im_livechat_channel`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`  
**Extended by packages:** `website_livechat`

Description: Livechat Channel

## Identity and behavior

- Mixins (classical inheritance): `rating.parent.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Channel Name | single line text |  | required |
| `button_text` | Text of the Button | single line text |  | default computed dynamically (_default_button_text); translatable |
| `default_message` | Welcome Message | single line text |  | default computed dynamically (_default_default_message); translatable; Help: This is an automated 'welcome' message that your visitor will see when they initiate a new conversation. |
| `header_background_color` | Header Background Color | single line text |  | default `#875A7B`; Help: Default background color of the channel header once open |
| `title_color` | Title Color | single line text |  | default `#FFFFFF`; Help: Default title color of the channel once open |
| `button_background_color` | Button Background Color | single line text |  | default `#875A7B`; Help: Default background color of the Livechat button |
| `button_text_color` | Button Text Color | single line text |  | default `#FFFFFF`; Help: Default text color of the Livechat button |
| `max_sessions_mode` | Sessions per Operator | selection |  | default `unlimited`; Help: If limited, operators will only handle the selected number of sessions at a time. |
| `max_sessions` | Maximum Sessions | integer |  | default `10`; Help: Maximum number of concurrent sessions per operator. |
| `block_assignment_during_call` | No Chats During Call | boolean |  | Help: While on a call, agents will not receive new conversations. |
| `review_link` | Review Link | single line text |  | Help: Visitors who leave a positive review will be redirected to this optional link. |
| `web_page` | Web Page | single line text |  | read only; computed by rule `_compute_web_page_link` (not stored); Help: URL to a static page where you client can discuss with the operator of the channel. |
| `are_you_inside` | Are you inside the matrix? | boolean |  | read only; computed by rule `_are_you_inside` (not stored) |
| `available_operator_ids` | Available Operator | many to many | `res.users` | computed by rule `_compute_available_operator_ids` (not stored) |
| `script_external` | Script (external) | rich text |  | read only; computed by rule `_compute_script_external` (not stored) |
| `nbr_channel` | Number of conversation | integer |  | read only; computed by rule `_compute_nbr_channel` (not stored) |
| `user_ids` | Agents | many to many | `res.users` | default computed dynamically (_default_user_ids); association table `im_livechat_channel_im_user` |
| `channel_ids` | Sessions | one to many | `discuss.channel` | inverse field `livechat_channel_id` |
| `chatbot_script_count` | Number of Chatbot | integer |  | computed by rule `_compute_chatbot_script_count` (not stored) |
| `rule_ids` | Rules | one to many | `im_livechat.channel.rule` | inverse field `channel_id` |
| `ongoing_session_count` | Number of Ongoing Sessions | integer |  | computed by rule `_compute_ongoing_sessions_count` (not stored) |
| `remaining_session_capacity` | Remaining Session Capacity | integer |  | computed by rule `_compute_remaining_session_capacity` (not stored) |

## Selection values

### `max_sessions_mode` (Sessions per Operator)

| Value | Label |
|---|---|
| `unlimited` | Unlimited |
| `limited` | Limited |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_max_sessions_mode_greater_than_zero` | Constraint | `CHECK(max_sessions > 0)` | Concurrent session number should be greater than zero. | `im_livechat` |

## Operations (29)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_user_ids` | preparation rule | self | `im_livechat` |  |  |
| `_default_button_text` | preparation rule | self | `im_livechat` |  |  |
| `_default_default_message` | preparation rule | self | `im_livechat` |  |  |
| `web_read` | lifecycle override | self, specification | `im_livechat` |  |  |
| `_are_you_inside` | internal rule | self | `im_livechat` |  |  |
| `_compute_ongoing_sessions_count` | computation | self | `im_livechat` | depends: `channel_ids.livechat_end_dt` |  |
| `_compute_remaining_session_capacity` | computation | self | `im_livechat` | depends: `block_assignment_during_call`, `max_sessions`, `user_ids.livechat_is_in_call`, `user_ids.livechat_ongoing_session_count` |  |
| `_compute_available_operator_ids` | computation | self | `im_livechat` | depends: `user_ids.channel_ids.last_interest_dt`, `user_ids.channel_ids.livechat_end_dt`, `user_ids.channel_ids.livechat_channel_id`, `user_ids.channel_ids.livechat_operator_id`, `user_ids.channel_member_ids`, `user_ids.im_status`, `user_ids.is_in_call`, `user_ids.partner_id` |  |
| `_check_review_link` | validation | self | `im_livechat` | constrains: `review_link` |  |
| `_get_available_operators_by_livechat_channel` | preparation rule | self, users | `im_livechat` |  | Return a dictionary mapping each livechat channel in ``self`` to the users that are available for that livechat channel, according to the user status and the optional limit of concurrent sessions of the livechat channel.  When ``users`` are provided, each user is attempted to be mapped for each livechat channel. Otherwise, only the users of each respective livechat channel are considered.  :param users: Optional list of users to consider. Every agent in ``self`` will be  considered if omitted. |
| `_get_ongoing_session_count_by_agent_livechat_channel` | preparation rule | self, users, filter_online | `im_livechat` |  | Return a dictionary mapping each ``(user, livechat_channel)`` pair to the number of ongoing livechat sessions.  :param users: List of users to consider for the session count. :param filter_online: If ``True``, only online agents will be considered. :type filter_online: bool :returns: A dictionary mapping ``(partner_id, livechat_channel_id)`` to the session count. :rtype: dict |
| `_compute_chatbot_script_count` | computation | self | `im_livechat` | depends: `rule_ids.chatbot_script_id` |  |
| `_compute_script_external` | computation | self | `im_livechat` |  |  |
| `_compute_web_page_link` | computation | self | `im_livechat` |  |  |
| `_compute_nbr_channel` | computation | self | `im_livechat` | depends: `channel_ids` |  |
| `action_join` | user action | self | `im_livechat` |  |  |
| `action_quit` | user action | self | `im_livechat` |  |  |
| `action_view_rating` | user action | self | `im_livechat` |  | Action to display the rating relative to the channel, so all rating of the sessions of the current channel :returns : the ir.action 'action_view_rating' with the correct context |
| `action_view_chatbot_scripts` | user action | self | `im_livechat` |  |  |
| `_get_livechat_discuss_channel_vals` | preparation rule | chatbot_script, agent, operator_partner, operator_model, **kwargs | `im_livechat`, `website_livechat` |  |  |
| `_get_agent_member_vals` | preparation rule | last_interest_dt, now, chatbot_script, operator_partner, operator_model, **kwargs | `im_livechat` |  |  |
| `_get_channel_name` | preparation rule | visitor_user, guest, agent, chatbot_script, operator_model, **kwargs | `im_livechat` |  |  |
| `_get_operator_info` | preparation rule | lang, country_id, previous_operator_id, chatbot_script_id, **kwargs | `im_livechat` |  |  |
| `_get_less_active_operator` | preparation rule | self, operator_statuses, operators | `im_livechat` |  | Retrieve the most available operator based on the following criteria: - Lowest number of active chats. - Not in  a call. - If an operator is in a call and has two or more active chats, don't   give priority over an operator with more conversations who is not in a   call.  :param operator_statuses: list of dictionaries containing the operator's     id, the number of active chats and a boolean indicating if the     operator is in a call. The list is ordered by the number of active     chats (ascending) and whether the operator is in a call     (descending). :param operators: recordset of :class: |
| `_get_operator` | preparation rule | self, previous_operator_id, lang, country_id, expertises, users | `im_livechat` |  | Return an operator for a livechat. Try to return the previous operator if available. If not, one of the most available operators be returned.  A livechat is considered 'active' if it has at least one message within the 30 minutes. This method will try to match the given lang, expertises and country_id.  (Some annoying conversions have to be made on the fly because this model holds 'res.users' as available operators and the discuss_channel model stores the partner_id of the randomly selected operator)  :param previous_operator_id: partner id of the previous operator with     whom the visitor wa |
| `_get_channel_infos` | preparation rule | self | `im_livechat` |  |  |
| `get_livechat_info` | operation | self, username | `im_livechat` |  |  |
| `_is_livechat_available` | internal rule | self | `im_livechat` |  |  |
| `create` | lifecycle override | self, vals_list | `website_livechat` | model_create_multi |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_review_link` | ValidationError | Invalid URL '%s'. The Review Link must start with 'http://' or 'https://'. | `im_livechat` |
| `action_join` | AccessError | Only Live Chat operators can join Live Chat channels | `im_livechat` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `im_livechat_group_user` | no | yes | no | no | `im_livechat` |
| `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `im_livechat.im_livechat_channel_view_kanban` | kanban |  | `are_you_inside`, `rating_count`, `name`, `nbr_channel`, `available_operator_ids`, `rating_percentage_satisfaction` | `action_quit`, `action_join` |  | `im_livechat` |
| `im_livechat.im_livechat_channel_view_form` | form |  | `are_you_inside`, `rating_count`, `chatbot_script_count`, `nbr_channel`, `rating_percentage_satisfaction`, `name`, `user_ids`, `partner_id`, `livechat_is_in_call`, `livechat_lang_ids`, `livechat_expertise_ids`, `livechat_ongoing_session_count`, `avatar_1024`, `name`, `livechat_lang_ids`, `livechat_expertise_ids`, `livechat_ongoing_session_count`, `livechat_is_in_call`, `ongoing_session_count`, `remaining_session_capacity`, `button_text`, `button_background_color`, `button_text_color`, `default_message`, `header_background_color`, `title_color`, `review_link`, `max_sessions_mode`, `max_sessions`, `block_assignment_during_call`, `rule_ids`, `script_external`, `web_page` | `Join Channel`, `Leave Channel`, `action_view_chatbot_scripts`, `%(discuss_channel_action_from_livechat_channel)d` |  | `im_livechat` |
| `im_livechat.im_livechat_channel_view_search` | search |  | `name` |  |  | `im_livechat` |
| `website_livechat.im_livechat_channel_view_form_add` | form |  | `name` |  |  | `website_livechat` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `im_livechat.im_livechat_channel_action` | Live Chat Channels | kanban,form |  |  |  | `im_livechat` |
| `website_livechat.im_livechat_channel_action_add` | New Channel | form |  | `{'create_from_website': True}` | new | `website_livechat` |

Machine-readable definition: `../../../schemas/data/entities/im_livechat.channel.json`; views: `../../../schemas/interfaces/views/im_livechat.channel.json`.
