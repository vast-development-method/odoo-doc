# Link between link previews and messages (`mail.message.link.preview`)

**Transport name:** `mail.message.link.preview`  
**Storage name:** `mail_message_link_preview`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Link between link previews and messages

## Identity and behavior

- Mixins (classical inheritance): `bus.listener.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `message_id` | Message | many to one | `mail.message` | required; indexed; on delete of the target: cascade |
| `link_preview_id` | Link Preview | many to one | `mail.link.preview` | required; indexed; on delete of the target: cascade |
| `sequence` | Sequence | integer |  |  |
| `is_hidden` | Is Hidden | boolean |  |  |
| `author_id` | Author | many to one |  | related through path `message_id.author_id` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_message_link_preview` | UniqueIndex | `(message_id, link_preview_id)` |  | `mail` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_bus_channel` | internal rule | self | `mail` |  |  |
| `_hide_and_notify` | internal rule | self | `mail` |  |  |
| `_unlink_and_notify` | internal rule | self | `mail` |  |  |
| `_to_store_defaults` | internal rule | self, target | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_erp_manager` | yes | yes | yes | yes | `mail` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.message_link_preview_list` | list |  | `author_id`, `is_hidden` |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.message.link.preview.json`; views: `../../../schemas/interfaces/views/mail.message.link.preview.json`.
