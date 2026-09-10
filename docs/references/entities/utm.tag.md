# campaign tracking parameter Tag (`utm.tag`)

**Transport name:** `utm.tag`  
**Storage name:** `utm_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `utm`

Description: UTM Tag

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `color` | Color Index | integer |  | default computed dynamically (lambda self: self._default_color()); Help: Tag color. No color means no display in kanban to distinguish internal tags from public categorization tags. |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `utm` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_color` | preparation rule | self | `utm` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_campaign` | yes | yes | yes | yes | `mass_mailing` |
| `base.group_user` | no | yes | no | no | `utm` |
| `base.group_system` | yes | yes | yes | yes | `utm` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `utm.utm_tag_view_tree` | list |  | `name` |  |  | `utm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `utm.action_view_utm_tag` | Campaign Tags |  |  |  |  | `utm` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mass_mailing.mass_mailing_tag_menu` |  | `mass_mailing_configuration` | `utm.action_view_utm_tag` | 2 | `mass_mailing.group_mass_mailing_campaign` |

Machine-readable definition: `../../../schemas/data/entities/utm.tag.json`; views: `../../../schemas/interfaces/views/utm.tag.json`.
