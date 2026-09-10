# Chatbot Message (`chatbot.message`)

**Transport name:** `chatbot.message`  
**Storage name:** `chatbot_message`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`

Description: Chatbot Message

## Identity and behavior

- Default ordering: `create_date desc, id desc`
- Display name field: `discuss_channel_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mail_message_id` | Related Mail Message | many to one | `mail.message` |  |
| `discuss_channel_id` | Discussion Channel | many to one | `discuss.channel` | required; indexed; on delete of the target: cascade |
| `script_step_id` | Chatbot Step | many to one | `chatbot.script.step` | indexed (btree_not_null) |
| `user_script_answer_id` | User's answer | many to one | `chatbot.script.answer` | on delete of the target: set null |
| `user_raw_script_answer_id` | User Raw Script Answer | integer |  | Help: Id of the script answer. Useful for statistics when answer is deleted. |
| `user_raw_answer` | User's raw answer | rich text |  |  |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_mail_message_id` | Constraint | `unique (mail_message_id)` | A mail.message can only be linked to a single chatbot message | `im_livechat` |
| `_channel_id_user_raw_script_answer_id_idx` | Index | `(discuss_channel_id, user_raw_script_answer_id) WHERE user_raw_script_answer_id IS NOT NULL` |  | `im_livechat` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |

Machine-readable definition: `../../../schemas/data/entities/chatbot.message.json`.
