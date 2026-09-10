# Link Tracker Code (`link.tracker.code`)

**Transport name:** `link.tracker.code`  
**Storage name:** `link_tracker_code`  
**Kind:** persistent entity (one table)  
**Defined by package:** `link_tracker`

Description: Link Tracker Code

## Identity and behavior

- Display name field: `code`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `code` | Short uniform resource locator Code | single line text |  | required |
| `link_id` | Link | many to one | `link.tracker` | required; indexed; on delete of the target: cascade |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_code` | Constraint | `unique( code )` | Code must be unique. | `link_tracker` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_random_code_strings` | preparation rule | self, n | `link_tracker` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `link_tracker` |
| `base.group_public` | no | no | no | no | `link_tracker` |
| `base.group_system` | yes | yes | yes | yes | `link_tracker` |
| `website.group_website_designer` | yes | yes | yes | yes | `website_links` |

Machine-readable definition: `../../../schemas/data/entities/link.tracker.code.json`.
