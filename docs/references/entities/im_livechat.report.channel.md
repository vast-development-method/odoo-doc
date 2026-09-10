# Livechat Support Channel Report (`im_livechat.report.channel`)

**Transport name:** `im_livechat.report.channel`  
**Storage name:** `im_livechat_report_channel`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`  
**Extended by packages:** `crm_livechat`

Description: Livechat Support Channel Report

## Identity and behavior

- Default ordering: `start_date, livechat_channel_id, channel_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (34)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `uuid` | UUID | single line text |  | read only |
| `channel_id` | Conversation | many to one | `discuss.channel` | read only |
| `channel_name` | Channel Name | single line text |  | read only |
| `livechat_channel_id` | Channel | many to one | `im_livechat.channel` | read only |
| `start_date` | Start Date of session | date and time |  | read only |
| `start_hour` | Start Hour of session | single line text |  | read only |
| `start_date_minutes` | Start Date of session, truncated to minutes | single line text |  | read only |
| `day_number` | Day of the Week | selection |  | read only |
| `time_to_answer` | Response Time | float |  | read only; precision `[16, 6]`; aggregated with avg; Help: Average time in hours to give the first answer to the visitor |
| `start_date_hour` | Hour of start Date of session | single line text |  | read only |
| `duration` | Duration (min) | float |  | read only; precision `[16, 2]`; aggregated with avg; Help: Duration of the conversation (in minutes) |
| `nbr_message` | Messages per Session | integer |  | read only; aggregated with avg; Help: Number of message in the conversation |
| `country_id` | Country of the visitor | many to one | `res.country` | read only |
| `lang_id` | Language | many to one | `res.lang` | read only; related through path `channel_id.livechat_lang_id` |
| `rating` | Rating | integer |  | read only; aggregated with avg |
| `rating_text` | Satisfaction Rate | single line text |  | read only |
| `partner_id` | Agent | many to one | `res.partner` | read only |
| `handled_by_bot` | Handled by Bot | integer |  | read only; aggregated with sum |
| `handled_by_agent` | Handled by Agent | integer |  | read only; aggregated with sum |
| `visitor_partner_id` | Customer | many to one | `res.partner` | read only |
| `call_duration_hour` | Call Duration | float |  | read only; precision `[16, 2]`; aggregated with avg |
| `has_call` | Whether the session had a call | float |  | read only |
| `number_of_calls` | # of Sessions with calls | float |  | read only; related through path `has_call`; aggregated with sum |
| `percentage_of_calls` | Session with Calls (%) | float |  | read only; related through path `has_call`; aggregated with avg |
| `session_outcome` | Session Outcome | selection |  | read only |
| `chatbot_script_id` | Chatbot | many to one | `chatbot.script` | read only |
| `chatbot_answers_path` | Chatbot Answers | single line text |  | read only |
| `chatbot_answers_path_str` | Chatbot Answers (String) | single line text |  | read only |
| `session_expertises` | Expertises used in this session (String) | single line text |  | read only |
| `session_expertise_ids` | Expertises used in this session | many to many | `im_livechat.expertise` | read only; related through path `channel_id.livechat_expertise_ids` |
| `conversation_tag_ids` | Tags used in this conversation | many to many | `im_livechat.conversation.tag` | read only; related through path `channel_id.livechat_conversation_tag_ids` |
| `agent_requesting_help_history` | Agent Requesting Help History | many to one | `im_livechat.channel.member.history` | read only; related through path `channel_id.livechat_agent_requesting_help_history` |
| `agent_providing_help_history` | Agent Providing Help History | many to one | `im_livechat.channel.member.history` | read only; related through path `channel_id.livechat_agent_providing_help_history` |
| `leads_created` | Leads created | integer |  | read only; aggregated with sum |

## Selection values

### `day_number` (Day of the Week)

| Value | Label |
|---|---|
| `0` | Sunday |
| `1` | Monday |
| `2` | Tuesday |
| `3` | Wednesday |
| `4` | Thursday |
| `5` | Friday |
| `6` | Saturday |

### `session_outcome` (Session Outcome)

| Value | Label |
|---|---|
| `no_answer` | Never Answered |
| `no_agent` | No one Available |
| `no_failure` | Success |
| `escalated` | Escalated |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_unknown_chatbot_answer_name` | internal rule | self | `im_livechat` |  |  |
| `_table_query` | internal rule | self | `im_livechat` |  |  |
| `_select` | internal rule | self | `crm_livechat`, `im_livechat` |  |  |
| `_from` | internal rule | self | `crm_livechat`, `im_livechat` |  |  |
| `_where` | internal rule | self | `im_livechat` |  |  |
| `formatted_read_group` | operation | self, domain, groupby, aggregates, having, offset, limit, order | `im_livechat` | model |  |
| `_read_group_orderby` | internal rule | self, order, groupby_terms, query | `im_livechat` |  |  |
| `action_open_discuss_channel_view` | user action | self, domain | `im_livechat` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `im_livechat_group_manager` | no | yes | no | no | `im_livechat` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_livechat.im_livechat_report_channel_view_search` | xpath | `im_livechat.im_livechat_report_channel_view_search` |  |  | `My Team` | `hr_livechat` |
| `im_livechat.im_livechat_report_channel_view_pivot` | pivot |  | `has_call`, `call_duration_hour`, `partner_id`, `percentage_of_calls`, `time_to_answer`, `duration`, `rating`, `number_of_calls` |  |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_view_list` | list |  | `start_date`, `channel_name`, `country_id`, `lang_id`, `session_expertise_ids`, `duration`, `nbr_message`, `rating_text` |  |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_view_form` | form |  | `channel_name` |  |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_view_graph` | graph |  | `call_duration_hour`, `has_call`, `percentage_of_calls`, `start_date`, `rating_text`, `rating` |  |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_view_search` | search |  | `partner_id`, `agent_requesting_help_history`, `agent_requesting_help_history`, `livechat_channel_id`, `country_id`, `chatbot_script_id`, `chatbot_answers_path_str`, `session_expertises`, `conversation_tag_ids`, `visitor_partner_id` |  | `My Sessions`, `Escalated`, `Not Answered`, `No one Available`, `Happy`, `Neutral`, `Unhappy`, `Date`, `Date: Last month`, `Date: Last week`, `Channel`, `Agent`, `group_by_agent_requesting_help`, `group_by_agent_providing_help`, `Rating`, `Country`, `Status`, `Chatbot`, `group_by_chatbot_answers`, `Expertise`, `Tags`, `group_by_customer`, `Hour of Day`, `Day of Week`, `Date` | `im_livechat` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `im_livechat.im_livechat_report_channel_action` | Sessions | graph,pivot |  | `{                     "search_default_filter_date_last_month": 1,                     "pivot_measures": ["__count", "time_to_answer", "duration", "rating", "number_of_calls"],                     "graph_measure": "__count__",                     "im_livechat_hide_partner_company": True,                 }` |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_time_to_answer_action` | Sessions | graph,pivot |  | `{"graph_measure": "time_to_answer", "search_default_filter_date_last_week":1}` |  | `im_livechat` |

Machine-readable definition: `../../../schemas/data/entities/im_livechat.report.channel.json`; views: `../../../schemas/interfaces/views/im_livechat.report.channel.json`.
