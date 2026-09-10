# customer relationship management Stages (`crm.stage`)

**Transport name:** `crm.stage`  
**Storage name:** `crm_stage`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm`

Description: CRM Stages

## Identity and behavior

- Default ordering: `sequence, name, id`
- Display name field: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Stage Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `1`; Help: Used to order stages. Lower is better. |
| `is_won` | Is Won Stage? | boolean |  |  |
| `rotting_threshold_days` | Days to rot | integer |  | default ; Help: Highlight opportunities that haven't been updated for this many days.         Set to 0 to disable. Changing this parameter will not affect the rotting status/date of resources last updated before this change. |
| `requirements` | Requirements | multi line text |  | Help: Enter here the internal requirements for this stage (ex: Offer sent to customer). It will appear as a tooltip over the stage's name. |
| `team_ids` | Sales Teams | many to many | `crm.team` | on delete of the target: restrict |
| `fold` | Folded in Pipeline | boolean |  | Help: This stage is folded in the kanban view when there are no records in that stage to display. |
| `team_count` | team_count | integer |  | computed by rule `_compute_team_count` (not stored) |
| `color` | Color | integer |  |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_team_count` | computation | self | `crm` | depends: `team_ids` |  |
| `_onchange_is_won` | on change | self | `crm` | onchange: `is_won` |  |
| `write` | lifecycle override | self, vals | `crm` |  | Since leads that are in a won stage must have their probability = 100%, this override ensures that setting a stage as won will set all the leads in that stage to probability = 100%. Inversely, if a won stage is not marked as won anymore, the lead probability should be recomputed based on automated probability. Note: If a user sets a stage as won and changes his mind right after, the manual probability will be lost in the process. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `crm` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `base.group_portal` | no | yes | no | no | `website_crm_partner_assign` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.crm_lead_stage_search` | search |  | `name`, `sequence`, `is_won`, `team_ids` |  |  | `crm` |
| `crm.crm_stage_tree` | list |  | `sequence`, `name`, `is_won`, `team_ids`, `rotting_threshold_days`, `color` |  |  | `crm` |
| `crm.crm_stage_form` | form |  | `name`, `fold`, `color`, `team_ids`, `is_won`, `rotting_threshold_days`, `team_count`, `requirements` |  |  | `crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.crm_stage_action` | Stages |  |  |  |  | `crm` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `crm.menu_crm_lead_stage_act` | Stages | `menu_crm_config_lead` | `crm.crm_stage_action` | 0 | `base.group_no_one` |

Machine-readable definition: `../../../schemas/data/entities/crm.stage.json`; views: `../../../schemas/interfaces/views/crm.stage.json`.
