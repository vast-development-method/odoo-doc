# Event Meeting Type (`calendar.event.type`)

**Transport name:** `calendar.event.type`  
**Storage name:** `calendar_event_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `calendar`

Description: Event Meeting Type

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `color` | Color | integer |  | default computed dynamically (_default_color) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `calendar` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_color` | preparation rule | self | `calendar` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `calendar` |
| `base.group_system` | yes | yes | yes | yes | `calendar` |
| `sales_team.group_sale_manager` | yes | yes | yes | no | `crm` |
| `base.group_user` | no | yes | no | no | `crm` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `group_hr_recruitment_user` | yes | yes | yes | no | `hr_recruitment` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `calendar.view_calendar_event_type_tree` | list |  | `name` |  |  | `calendar` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `calendar.action_calendar_event_type` | Meeting Types |  |  |  |  | `calendar` |

Machine-readable definition: `../../../schemas/data/entities/calendar.event.type.json`; views: `../../../schemas/interfaces/views/calendar.event.type.json`.
