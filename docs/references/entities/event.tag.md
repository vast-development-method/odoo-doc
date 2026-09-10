# Event Tag (`event.tag`)

**Transport name:** `event.tag`  
**Storage name:** `event_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `website_event`

Description: Event Tag

## Identity and behavior

- Mixins (classical inheritance): `website.published.multi.mixin`
- Default ordering: `category_sequence, sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default  |
| `category_id` | Category | many to one | `event.tag.category` | required; indexed; on delete of the target: cascade |
| `category_sequence` | Category Sequence | integer |  | related through path `category_id.sequence` and stored |
| `color` | Color Index | integer |  | default computed dynamically (lambda self: self._default_color()); Help: Tag color. No color means no display in kanban or front-end, to distinguish internal tags from public categorization tags. |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_color` | preparation rule | self | `event` |  |  |
| `default_get` | lifecycle override | self, fields | `website_event` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `event` |
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_user` | yes | yes | yes | no | `event` |
| `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `base.group_public` | no | yes | no | no | `website_event` |
| `base.group_portal` | no | yes | no | no | `website_event` |
| `base.group_user` | no | yes | no | no | `website_event` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Tag: public/portal: color = published and category = published | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('category_id.website_published', '=', True), ('color', '!=', False), ('color', '!=', 0)]` | True | False | False | False |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.event_tag_view_tree` | list |  | `sequence`, `name`, `category_id`, `color` |  |  | `event` |
| `event.event_tag_view_form` | form |  | `name`, `category_id`, `color` |  |  | `event` |
| `website_event.event_tag_view_form_inherit` | xpath | `event.event_tag_view_form` | `website_id` |  |  | `website_event` |

Machine-readable definition: `../../../schemas/data/entities/event.tag.json`; views: `../../../schemas/interfaces/views/event.tag.json`.
