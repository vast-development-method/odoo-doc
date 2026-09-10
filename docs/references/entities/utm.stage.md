# Campaign Stage (`utm.stage`)

**Transport name:** `utm.stage`  
**Storage name:** `utm_stage`  
**Kind:** persistent entity (one table)  
**Defined by package:** `utm`

Description: Campaign Stage

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `1` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `base.group_user` | no | yes | no | no | `utm` |
| `base.group_system` | yes | yes | yes | yes | `utm` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `utm.utm_stage_view_search` | search |  | `name` |  |  | `utm` |
| `utm.utm_stage_view_tree` | list |  | `sequence`, `name` |  |  | `utm` |
| `utm.utm_stage_view_form` | form |  | `name` |  |  | `utm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `utm.action_view_utm_stage` | UTM Stages | list,form |  |  |  | `utm` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mass_mailing.menu_view_mass_mailing_stages` | Campaign Stages | `mass_mailing_configuration` | `utm.action_view_utm_stage` | 1 | `mass_mailing.group_mass_mailing_campaign` |

Machine-readable definition: `../../../schemas/data/entities/utm.stage.json`; views: `../../../schemas/interfaces/views/utm.stage.json`.
