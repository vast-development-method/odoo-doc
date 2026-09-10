# Digest Tips (`digest.tip`)

**Transport name:** `digest.tip`  
**Storage name:** `digest_tip`  
**Kind:** persistent entity (one table)  
**Defined by package:** `digest`

Description: Digest Tips

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `1`; Help: Used to display digest tip in email template base on order |
| `name` | Name | single line text |  | translatable |
| `user_ids` | Recipients | many to many | `res.users` | Help: Users having already received this tip |
| `tip_description` | Tip description | rich text |  | translatable |
| `group_id` | Authorized Group | many to one | `res.groups` | default computed dynamically (lambda self: self.env.ref('base.group_user')) |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_erp_manager` | yes | yes | yes | yes | `digest` |
| `base.group_user` | no | yes | no | no | `digest` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `digest.digest_tip_view_tree` | list |  | `sequence`, `name`, `group_id` |  |  | `digest` |
| `digest.digest_tip_view_form` | form |  | `name`, `tip_description`, `group_id`, `user_ids` |  |  | `digest` |
| `digest.digest_tip_view_search` | search |  | `name`, `tip_description`, `group_id` |  |  | `digest` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `digest.digest_tip_action` | Digest Tips |  |  |  |  | `digest` |

Machine-readable definition: `../../../schemas/data/entities/digest.tip.json`; views: `../../../schemas/interfaces/views/digest.tip.json`.
