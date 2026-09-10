# Live Chat Conversation Tags (`im_livechat.conversation.tag`)

**Transport name:** `im_livechat.conversation.tag`  
**Storage name:** `im_livechat_conversation_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`

Description: Live Chat Conversation Tags

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `color` | Color | integer |  | default computed dynamically (_get_default_color) |
| `conversation_ids` | Discuss Channels | many to many | `discuss.channel` | association table `livechat_conversation_tag_rel` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_unique` | UniqueIndex | `(name)` |  | `im_livechat` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `im_livechat` | model |  |
| `_unlink_sync_conversation` | internal rule | self | `im_livechat` | ondelete |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `im_livechat_group_user` | yes | yes | yes | no | `im_livechat` |
| `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `im_livechat.im_livechat_channel_conversation_tag_view_list` | list |  | `name`, `color` |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_conversation_tag_view_form` | form |  | `name`, `color` |  |  | `im_livechat` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `im_livechat.livechat_conversation_tag_action` | Tags | list,form |  |  |  | `im_livechat` |

Machine-readable definition: `../../../schemas/data/entities/im_livechat.conversation.tag.json`; views: `../../../schemas/interfaces/views/im_livechat.conversation.tag.json`.
