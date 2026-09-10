# Message Translation (`mail.message.translation`)

**Transport name:** `mail.message.translation`  
**Storage name:** `mail_message_translation`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Message Translation

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `message_id` | Message | many to one | `mail.message` | required; on delete of the target: cascade |
| `source_lang` | Source Language | single line text |  | required; Help: Result of the language detection based on its content. |
| `target_lang` | Target Language | single line text |  | required; Help: Shortened language code used as the target for the translation request. |
| `body` | Translation Body | rich text |  | required; Help: String received from the translation request. |
| `create_date` | Create Date | date and time |  | indexed |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique` | UniqueIndex | `(message_id, target_lang)` |  | `mail` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_gc_translations` | background operation | self | `mail` | autovacuum |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.message.translation.json`.
