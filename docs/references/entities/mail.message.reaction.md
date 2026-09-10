# Message Reaction (`mail.message.reaction`)

**Transport name:** `mail.message.reaction`  
**Storage name:** `mail_message_reaction`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Message Reaction

## Identity and behavior

- Default ordering: `id desc`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `message_id` | Message | many to one | `mail.message` | required; read only; indexed; on delete of the target: cascade |
| `content` | Content | single line text |  | required; read only |
| `partner_id` | Reacting Partner | many to one | `res.partner` | read only; on delete of the target: cascade |
| `guest_id` | Reacting Guest | many to one | `mail.guest` | read only; on delete of the target: cascade |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_partner_unique` | UniqueIndex | `(message_id, content, partner_id) WHERE partner_id IS NOT NULL` |  | `mail` |
| `_guest_unique` | UniqueIndex | `(message_id, content, guest_id) WHERE guest_id IS NOT NULL` |  | `mail` |
| `_partner_or_guest_exists` | Constraint | `CHECK((partner_id IS NOT NULL AND guest_id IS NULL) OR (partner_id IS NULL AND guest_id IS NOT NULL))` | A message reaction must be from a partner or from a guest. | `mail` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_to_store` | internal rule | self, store, fields | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_message_reaction_view_form` | form |  | `message_id`, `content`, `partner_id`, `guest_id` |  |  | `mail` |
| `mail.mail_message_reaction_view_tree` | list |  | `id`, `message_id`, `content`, `partner_id`, `guest_id` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_message_reaction_action` | Message Reactions | list,form |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.message.reaction.json`; views: `../../../schemas/interfaces/views/mail.message.reaction.json`.
