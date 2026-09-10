# Event Sponsor Level (`event.sponsor.type`)

**Transport name:** `event.sponsor.type`  
**Storage name:** `event_sponsor_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_exhibitor`

Description: Event Sponsor Level

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Sponsor Level | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default computed dynamically (_default_sequence) |
| `display_ribbon_style` | Ribbon Style | selection |  | default `no_ribbon` |

## Selection values

### `display_ribbon_style` (Ribbon Style)

| Value | Label |
|---|---|
| `no_ribbon` | No Ribbon |
| `Gold` | Gold |
| `Silver` | Silver |
| `Bronze` | Bronze |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_sequence` | preparation rule | self | `website_event_exhibitor` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_manager` | yes | yes | yes | yes | `website_event_exhibitor` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_exhibitor.event_sponsor_type_view_form` | form |  | `name`, `display_ribbon_style`, `sequence` |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_type_view_tree` | list |  | `sequence`, `name`, `display_ribbon_style` |  |  | `website_event_exhibitor` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_exhibitor.event_sponsor_type_action` | Sponsor Levels |  |  |  |  | `website_event_exhibitor` |

Machine-readable definition: `../../../schemas/data/entities/event.sponsor.type.json`; views: `../../../schemas/interfaces/views/event.sponsor.type.json`.
