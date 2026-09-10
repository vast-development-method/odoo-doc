# Website Event Menu (`website.event.menu`)

**Transport name:** `website.event.menu`  
**Storage name:** `website_event_menu`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event`  
**Extended by packages:** `website_event_track`, `website_event_booth`, `website_event_exhibitor`

Description: Website Event Menu

## Identity and behavior

- Mixins (classical inheritance): `website.seo.metadata`
- Display name field: `menu_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `menu_id` | Menu | many to one | `website.menu` | on delete of the target: cascade |
| `event_id` | Event | many to one | `event.event` | indexed (btree_not_null); on delete of the target: cascade |
| `view_id` | View | many to one | `ir.ui.view` | on delete of the target: cascade; Help: Used when not being an url based menu |
| `menu_type` | Menu Type | selection |  | required; on delete of the target: {"exhibitor": "cascade"}; extended by packages `website_event_track`, `website_event_booth`, `website_event_exhibitor` |

## Selection values

### `menu_type` (Menu Type)

| Value | Label |
|---|---|
| `community` | Community Menu |
| `introduction` | Home |
| `register` | Practical |
| `other` | Other |
| `track` | Event Tracks Menus |
| `track_proposal` | Event Proposals Menus |
| `booth` | Event Booth Menus |
| `exhibitor` | Exhibitors Menus |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `copy` | lifecycle override | self, default | `website_event` |  |  |
| `_copy_children_views` | internal rule | self, new_view, children_views, website_id | `website_event` | model | Duplicate the children associated in the new view |
| `unlink` | lifecycle override | self | `website_event` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_event` |
| `base.group_portal` | no | yes | no | no | `website_event` |
| `base.group_user` | no | yes | no | no | `website_event` |
| `event.group_event_user` | yes | yes | yes | yes | `website_event` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event.website_event_menu_view_search` | search |  | `menu_id`, `event_id`, `menu_type`, `view_id` |  |  | `website_event` |
| `website_event.website_event_menu_view_form` | form |  | `menu_id`, `event_id`, `menu_type`, `view_id` |  |  | `website_event` |
| `website_event.website_event_menu_view_tree` | list |  | `menu_id`, `event_id`, `menu_type`, `view_id` |  |  | `website_event` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event.website_event_menu_action` | Menus | list,form |  | `{'create': False}` |  | `website_event` |

Machine-readable definition: `../../../schemas/data/entities/website.event.menu.json`; views: `../../../schemas/interfaces/views/website.event.menu.json`.
