# Rank based on karma (`gamification.karma.rank`)

**Transport name:** `gamification.karma.rank`  
**Storage name:** `gamification_karma_rank`  
**Kind:** persistent entity (one table)  
**Defined by package:** `gamification`

Description: Rank based on karma

## Identity and behavior

- Mixins (classical inheritance): `image.mixin`
- Default ordering: `karma_min`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Rank Name | multi line text |  | required; translatable |
| `description` | Description | rich text |  | translatable |
| `description_motivational` | Motivational | rich text |  | translatable; Help: Motivational phrase to reach this rank on your profile page |
| `karma_min` | Required Karma | integer |  | required; default `1` |
| `user_ids` | Users | one to many | `res.users` | inverse field `rank_id` |
| `rank_users_count` | # Users | integer |  | computed by rule `_compute_rank_users_count` (not stored) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_karma_min_check` | Constraint | `CHECK( karma_min > 0 )` | The required karma has to be above 0. | `gamification` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_rank_users_count` | computation | self | `gamification` | depends: `user_ids` |  |
| `create` | lifecycle override | self, vals_list | `gamification` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `gamification` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `gamification` |
| `base.group_portal` | no | yes | no | no | `gamification` |
| `base.group_user` | no | yes | no | no | `gamification` |
| `base.group_system` | yes | yes | yes | yes | `gamification` |
| `website.group_website_restricted_editor` | yes | yes | yes | yes | `website_profile` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.gamification_karma_ranks_view_search` | search |  | `name`, `karma_min`, `description`, `user_ids` |  |  | `gamification` |
| `gamification.gamification_karma_ranks_view_tree` | list |  | `name`, `karma_min`, `rank_users_count` |  |  | `gamification` |
| `gamification.gamification_karma_rank_view_form` | form |  | `rank_users_count`, `image_1920`, `name`, `karma_min`, `create_date`, `description`, `description_motivational` | `%(action_current_rank_users)d` |  | `gamification` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `gamification.gamification_karma_ranks_action` | Ranks | list,form |  |  |  | `gamification` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `website_forum.menu_forum_rank_global` | Ranks | `menu_website_forum_global` | `gamification.gamification_karma_ranks_action` | 20 |  |

Machine-readable definition: `../../../schemas/data/entities/gamification.karma.rank.json`; views: `../../../schemas/interfaces/views/gamification.karma.rank.json`.
