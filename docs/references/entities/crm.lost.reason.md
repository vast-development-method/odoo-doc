# Opp. Lost Reason (`crm.lost.reason`)

**Transport name:** `crm.lost.reason`  
**Storage name:** `crm_lost_reason`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm`

Description: Opp. Lost Reason

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True` |
| `leads_count` | Leads Count | integer |  | computed by rule `_compute_leads_count` (not stored) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_leads_count` | computation | self | `crm` |  |  |
| `action_lost_leads` | user action | self | `crm` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `base.group_user` | no | yes | no | no | `crm` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.crm_lost_reason_view_search` | search |  | `name` |  | `Active`, `Archived` | `crm` |
| `crm.crm_lost_reason_view_form` | form |  | `leads_count`, `name`, `active` | `action_lost_leads` |  | `crm` |
| `crm.crm_lost_reason_view_tree` | list |  | `name` |  |  | `crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.crm_lost_reason_action` | Lost Reasons | list,form |  |  |  | `crm` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `crm.menu_crm_lost_reason` | Lost Reasons | `menu_crm_config_lead` | `crm.crm_lost_reason_action` | 6 |  |

Machine-readable definition: `../../../schemas/data/entities/crm.lost.reason.json`; views: `../../../schemas/interfaces/views/crm.lost.reason.json`.
