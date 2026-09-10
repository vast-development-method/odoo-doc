# Live Chat Expertise (`im_livechat.expertise`)

**Transport name:** `im_livechat.expertise`  
**Storage name:** `im_livechat_expertise`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`

Description: Live Chat Expertise

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `user_ids` | Operators | many to many | `res.users` | computed by rule `_compute_user_ids` (not stored); writable through an inverse rule |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_unique` | UniqueIndex | `(name)` |  | `im_livechat` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_user_ids` | computation | self | `im_livechat` |  |  |
| `_inverse_user_ids` | inverse computation | self | `im_livechat` |  |  |
| `_get_users_by_expertise` | preparation rule | self | `im_livechat` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `im_livechat` |
| `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `im_livechat.im_livechat_expertise_view_list` | list |  | `name`, `user_ids` |  |  | `im_livechat` |
| `im_livechat.im_livechat_expertise_view_form` | form |  | `name`, `user_ids` |  |  | `im_livechat` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `im_livechat.expertise_action` | Expertise | list,form |  |  |  | `im_livechat` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `im_livechat.expertise_menu` | Expertise | `livechat_config` | `im_livechat.expertise_action` | 25 |  |

Machine-readable definition: `../../../schemas/data/entities/im_livechat.expertise.json`; views: `../../../schemas/interfaces/views/im_livechat.expertise.json`.
