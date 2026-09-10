# Keep the channel member history (`im_livechat.channel.member.history`)

**Transport name:** `im_livechat.channel.member.history`  
**Storage name:** `im_livechat_channel_member_history`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`

Description: Keep the channel member history

## Identity and behavior

- Display name search fields: `["partner_id", "guest_id"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (26)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `member_id` | Member | many to one | `discuss.channel.member` | indexed (btree_not_null) |
| `livechat_member_type` | Livechat Member Type | selection |  | computed by rule `_compute_member_fields` and stored |
| `channel_id` | Channel | many to one | `discuss.channel` | computed by rule `_compute_member_fields` and stored; indexed; on delete of the target: cascade |
| `guest_id` | Guest | many to one | `mail.guest` | computed by rule `_compute_member_fields` and stored; indexed (btree_not_null) |
| `partner_id` | Partner | many to one | `res.partner` | computed by rule `_compute_member_fields` and stored; indexed (btree_not_null) |
| `chatbot_script_id` | Chatbot Script | many to one | `chatbot.script` | computed by rule `_compute_member_fields` and stored; indexed (btree_not_null) |
| `agent_expertise_ids` | Agent Expertise | many to many | `im_livechat.expertise` | computed by rule `_compute_member_fields` and stored |
| `conversation_tag_ids` | Conversation Tag | many to many | `im_livechat.conversation.tag` | related through path `channel_id.livechat_conversation_tag_ids`; association table `im_livechat_channel_member_history_conversation_tag_rel` |
| `avatar_128` | Avatar 128 | binary |  | computed by rule `_compute_avatar_128` (not stored) |
| `session_country_id` | Session Country | many to one | `res.country` | related through path `channel_id.country_id` |
| `session_livechat_channel_id` | Live chat channel | many to one | `im_livechat.channel` | related through path `channel_id.livechat_channel_id` |
| `session_outcome` | Session Outcome | selection |  | related through path `channel_id.livechat_outcome` |
| `session_start_hour` | Session Start Hour | float |  | related through path `channel_id.livechat_start_hour` |
| `session_week_day` | Session Week Day | selection |  | related through path `channel_id.livechat_week_day` |
| `session_duration_hour` | Session Duration | float |  | computed by rule `_compute_session_duration_hour` and stored; aggregated with avg; Help: Time spent by the persona in the session in hours |
| `rating_id` | Rating | many to one | `rating.rating` | computed by rule `_compute_rating_id` and stored |
| `rating` | Rating | float |  | related through path `rating_id.rating` |
| `rating_text` | Rating text | selection |  | related through path `rating_id.rating_text` |
| `call_history_ids` | Call History | many to many | `discuss.call.history` |  |
| `has_call` | Has Call | float |  | computed by rule `_compute_has_call` and stored |
| `call_count` | # of Sessions with Calls | float |  | related through path `has_call`; aggregated with sum |
| `call_percentage` | Session with Calls (%) | float |  | related through path `has_call`; aggregated with avg |
| `call_duration_hour` | Call Duration | float |  | computed by rule `_compute_call_duration_hour` and stored; aggregated with sum |
| `message_count` | # of Messages per Session | integer |  | aggregated with avg |
| `help_status` | Help Status | selection |  | computed by rule `_compute_help_status` and stored |
| `response_time_hour` | Response Time | float |  | aggregated with avg |

## Selection values

### `livechat_member_type` (Livechat Member Type)

| Value | Label |
|---|---|
| `agent` | Agent |
| `visitor` | Visitor |
| `bot` | Chatbot |

### `help_status` (Help Status)

| Value | Label |
|---|---|
| `requested` | Help Requested |
| `provided` | Help Provided |

## State fields

State machine fields of this entity: `help_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (4)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_member_id_unique` | Constraint | `UNIQUE(member_id)` | Members can only be linked to one history | `im_livechat` |
| `_channel_id_partner_id_unique` | UniqueIndex | `(channel_id, partner_id) WHERE partner_id IS NOT NULL` | One partner can only be linked to one history on a channel | `im_livechat` |
| `_channel_id_guest_id_unique` | UniqueIndex | `(channel_id, guest_id) WHERE guest_id IS NOT NULL` | One guest can only be linked to one history on a channel | `im_livechat` |
| `_partner_id_or_guest_id_constraint` | Constraint | `CHECK(partner_id IS NULL OR guest_id IS NULL)` | History should either be linked to a partner or a guest but not both | `im_livechat` |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_constraint_channel_id` | validation | self | `im_livechat` | constrains: `channel_id` |  |
| `_compute_member_fields` | computation | self | `im_livechat` | depends: `member_id` |  |
| `_compute_display_name` | computation | self | `im_livechat` | depends: `livechat_member_type`, `partner_id.name`, `partner_id.display_name`, `guest_id.name` |  |
| `_compute_avatar_128` | computation | self | `im_livechat` | depends: `partner_id.avatar_128`, `guest_id.avatar_128` |  |
| `_compute_has_call` | computation | self | `im_livechat` | depends: `call_history_ids` |  |
| `_compute_call_duration_hour` | computation | self | `im_livechat` | depends: `call_history_ids.duration_hour` |  |
| `_compute_help_status` | computation | self | `im_livechat` | depends: `channel_id.livechat_agent_requesting_help_history`, `channel_id.livechat_agent_providing_help_history` |  |
| `_compute_rating_id` | computation | self | `im_livechat` | depends: `channel_id.rating_ids` |  |
| `_compute_session_duration_hour` | computation | self | `im_livechat` | depends: `create_date`, `channel_id.livechat_end_dt`, `channel_id.message_ids` |  |
| `action_open_discuss_channel_view` | user action | self, domain | `im_livechat` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constraint_channel_id` | ValidationError | Cannot create history as it is only available for live chats: %(histories)s. | `im_livechat` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `im_livechat_group_user` | no | yes | no | no | `im_livechat` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_livechat.website_livechat_agent_history_view_search` | xpath | `im_livechat.im_livechat_agent_history_view_search` |  |  | `My Team` | `hr_livechat` |
| `im_livechat.im_livechat_channel_member_history_view_search` | search |  | `channel_id`, `partner_id`, `chatbot_script_id`, `guest_id`, `livechat_member_type` |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_member_history_view_tree` | list |  | `create_date`, `channel_id`, `partner_id`, `chatbot_script_id`, `guest_id`, `session_duration_hour`, `livechat_member_type` |  |  | `im_livechat` |
| `im_livechat.im_livechat_agent_history_view_search` | search |  | `partner_id`, `session_livechat_channel_id`, `session_country_id`, `agent_expertise_ids`, `conversation_tag_ids` |  | `My Sessions`, `Escalated`, `Not Answered`, `Happy`, `Neutral`, `Unhappy`, `Date`, `Date: Last month`, `Date: Last week`, `Channel`, `Agent`, `group_by_help_status`, `Rating`, `Country`, `Status`, `Expertise`, `Tags`, `Hour of Day`, `Day of Week`, `Date` | `im_livechat` |
| `im_livechat.im_livechat_agent_history_view_graph` | graph |  | `has_call`, `session_start_hour`, `session_week_day`, `call_duration_hour`, `call_percentage`, `rating`, `response_time_hour`, `session_duration_hour` |  |  | `im_livechat` |
| `im_livechat.im_livechat_agent_history_view_pivot` | pivot |  | `has_call`, `session_start_hour`, `session_week_day`, `partner_id`, `call_duration_hour`, `call_percentage`, `response_time_hour`, `session_duration_hour`, `rating`, `call_count` |  |  | `im_livechat` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `im_livechat.im_livechat_channel_member_history_action` | Member History | list,form |  | `{"create": False}` |  | `im_livechat` |
| `im_livechat.im_livechat_agent_history_action` | Agents | pivot,graph | `[('livechat_member_type', '=', 'agent')]` | `{                     "search_default_group_by_agent": 1,                     "search_default_filter_date_last_month": 1,                     "graph_measure": "__count__",                     "pivot_measures": ["__count", "response_time_hour", "session_duration_hour", "rating", "call_count"],                     "im_livechat_hide_partner_company": True                 }` |  | `im_livechat` |

Machine-readable definition: `../../../schemas/data/entities/im_livechat.channel.member.history.json`; views: `../../../schemas/interfaces/views/im_livechat.channel.member.history.json`.
