# Mail Bot (`mail.bot`)

**Transport name:** `mail.bot`  
**Storage name:** `mail_bot`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail_bot`

Description: Mail Bot

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_apply_logic` | internal rule | self, channel, values, command | `mail_bot` |  | Apply bot logic to generate an answer (or not) for the user The logic will only be applied if odoobot is in a chat with a user or if someone pinged odoobot.   :param channel: the discuss channel where the user message was posted/odoobot will answer.  :param values: msg_values of the message_post or other values needed by logic  :param command: the name of the called command if the logic is not triggered by a message_post |
| `_get_style_dict` | preparation rule |  | `mail_bot` |  |  |
| `_get_answer` | preparation rule | self, channel, body, values, command | `mail_bot` |  |  |
| `_body_contains_emoji` | internal rule | self, body | `mail_bot` |  |  |
| `_is_help_requested` | internal rule | self, body | `mail_bot` |  | Returns whether a message linking to the documentation and videos should be sent back to the user. |

Machine-readable definition: `../../../schemas/data/entities/mail.bot.json`.
