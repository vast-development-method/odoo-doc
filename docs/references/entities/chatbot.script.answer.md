# Chatbot Script Answer (`chatbot.script.answer`)

**Transport name:** `chatbot.script.answer`  
**Storage name:** `chatbot_script_answer`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`

Description: Chatbot Script Answer

## Identity and behavior

- Default ordering: `script_step_id, sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Answer | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `1` |
| `redirect_link` | Redirect Link | single line text |  | Help: The visitor will be redirected to this link upon clicking the option (note that the script will end if the link is external to the livechat website). |
| `script_step_id` | Script Step | many to one | `chatbot.script.step` | required; indexed; on delete of the target: cascade |
| `chatbot_script_id` | Chatbot Script | many to one |  | related through path `script_step_id.chatbot_script_id` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `im_livechat` | depends: `script_step_id`; depends_context: `chatbot_script_answer_display_short_name` |  |
| `_search_display_name` | search rule | self, operator, value | `im_livechat` | model | Search the records whose name or step message are matching the `name` pattern. |
| `_to_store_defaults` | internal rule | self, target | `im_livechat` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `im_livechat` |
| `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `im_livechat.chatbot_script_answer_view_form` | form |  | `name`, `script_step_id` |  |  | `im_livechat` |
| `im_livechat.chatbot_script_answer_view_tree` | list |  | `sequence`, `script_step_id`, `name`, `redirect_link` |  |  | `im_livechat` |

Machine-readable definition: `../../../schemas/data/entities/chatbot.script.answer.json`; views: `../../../schemas/interfaces/views/chatbot.script.answer.json`.
