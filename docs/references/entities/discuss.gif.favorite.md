# Save favorite GIF from Tenor application programming interface (`discuss.gif.favorite`)

**Transport name:** `discuss.gif.favorite`  
**Storage name:** `discuss_gif_favorite`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Save favorite GIF from Tenor API

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `tenor_gif_id` | GIF id from Tenor | single line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_user_gif_favorite` | Constraint | `unique(create_uid,tenor_gif_id)` | User should not have duplicated favorite GIF | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Discuss.gif.favorite: User access | `[Command.link(ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Discuss.gif.favorite: admin full access | `[Command.link(ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.discuss_gif_favorite_view_form` | form |  | `id`, `tenor_gif_id`, `create_uid` |  |  | `mail` |
| `mail.discuss_gif_favorite_view_tree` | list |  | `id`, `tenor_gif_id`, `create_uid` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.discuss_gif_favorite_action` | GIF favorite | list,form |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/discuss.gif.favorite.json`; views: `../../../schemas/interfaces/views/discuss.gif.favorite.json`.
