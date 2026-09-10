# Event Tag Category (`event.tag.category`)

**Transport name:** `event.tag.category`  
**Storage name:** `event_tag_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `website_event`

Description: Event Tag Category

## Identity and behavior

- Mixins (classical inheritance): `website.published.multi.mixin`
- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default computed dynamically (_default_sequence) |
| `tag_ids` | Tags | one to many | `event.tag` | inverse field `category_id` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_sequence` | preparation rule | self | `event` |  | Here we use a _default method instead of ordering on 'sequence, id' to prevent adding a new related stored field in the 'event.tag' model that would hold the category id. |
| `_default_is_published` | preparation rule | self | `website_event` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_user` | yes | yes | yes | yes | `event` |
| `base.group_public` | no | yes | no | no | `website_event` |
| `base.group_portal` | no | yes | no | no | `website_event` |
| `base.group_user` | no | yes | no | no | `website_event` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.event_tag_category_view_tree` | list |  | `sequence`, `name`, `tag_ids` |  |  | `event` |
| `event.event_tag_category_view_form` | form |  | `name`, `tag_ids`, `sequence`, `name`, `color` |  |  | `event` |
| `website_event.event_tag_category_view_form` | field | `event.event_tag_category_view_form` | `tag_ids`, `is_published`, `website_id` |  |  | `website_event` |
| `website_event.event_tag_category_view_tree` | field | `event.event_tag_category_view_tree` | `tag_ids`, `is_published` |  |  | `website_event` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event.event_tag_category_action_tree` | Event Tags Categories | list,form |  |  |  | `event` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `event.menu_event_category` |  |  | `event.event_tag_category_action_tree` |  |  |

Machine-readable definition: `../../../schemas/data/entities/event.tag.category.json`; views: `../../../schemas/interfaces/views/event.tag.category.json`.
