# Post Vote (`forum.post.vote`)

**Transport name:** `forum.post.vote`  
**Storage name:** `forum_post_vote`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_forum`

Description: Post Vote

## Identity and behavior

- Default ordering: `create_date desc, id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `post_id` | Post | many to one | `forum.post` | required; indexed; on delete of the target: cascade |
| `user_id` | User | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.uid); on delete of the target: cascade |
| `vote` | Vote | selection |  | required; default `1` |
| `create_date` | Create Date | date and time |  | read only; indexed |
| `forum_id` | Forum | many to one | `forum.forum` | related through path `post_id.forum_id` and stored; indexed (btree_not_null) |
| `recipient_id` | To | many to one | `res.users` | related through path `post_id.create_uid` and stored |

## Selection values

### `vote` (Vote)

| Value | Label |
|---|---|
| `1` | 1 |
| `-1` | -1 |
| `0` | 0 |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_vote_uniq` | Constraint | `unique (post_id, user_id)` | Vote already exists! | `website_forum` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_karma_value` | preparation rule | self, old_vote, new_vote, up_karma, down_karma | `website_forum` |  | Return the karma to add / remove based on the old vote and on the new vote. |
| `create` | lifecycle override | self, vals_list | `website_forum` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website_forum` |  |  |
| `_check_general_rights` | validation | self, vals | `website_forum` |  |  |
| `_check_karma_rights` | validation | self, upvote | `website_forum` |  |  |
| `_vote_update_karma` | internal rule | self, old_vote, new_vote | `website_forum` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_general_rights` | UserError | It is not allowed to vote for its own post. | `website_forum` |
| `_check_general_rights` | UserError | It is not allowed to modify someone else's vote. | `website_forum` |
| `_check_karma_rights` | AccessError | %d karma required to upvote. | `website_forum` |
| `_check_karma_rights` | AccessError | %d karma required to downvote. | `website_forum` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | yes | yes | yes | no | `website_forum` |
| `base.group_user` | yes | yes | yes | yes | `website_forum` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Website forum vote: own votes only | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Website forum vote: all votes | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/forum.post.vote.json`.
