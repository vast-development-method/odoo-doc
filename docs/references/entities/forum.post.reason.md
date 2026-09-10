# Post Closing Reason (`forum.post.reason`)

**Transport name:** `forum.post.reason`  
**Storage name:** `forum_post_reason`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_forum`

Description: Post Closing Reason

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Closing Reason | single line text |  | required; translatable |
| `reason_type` | Reason Type | selection |  | default `basic` |

## Selection values

### `reason_type` (Reason Type)

| Value | Label |
|---|---|
| `basic` | Basic |
| `offensive` | Offensive |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_forum` |
| `base.group_portal` | no | yes | no | no | `website_forum` |
| `base.group_user` | yes | yes | yes | yes | `website_forum` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_forum.forum_post_reason_view_list` | list |  | `name`, `reason_type` |  |  | `website_forum` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_forum.forum_post_reason_action` | Post Close Reason | list |  |  |  | `website_forum` |

Machine-readable definition: `../../../schemas/data/entities/forum.post.reason.json`; views: `../../../schemas/interfaces/views/forum.post.reason.json`.
