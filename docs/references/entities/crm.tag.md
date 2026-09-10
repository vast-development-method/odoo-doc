# customer relationship management Tag (`crm.tag`)

**Transport name:** `crm.tag`  
**Storage name:** `crm_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sales_team`

Description: CRM Tag

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Tag Name | single line text |  | required; translatable |
| `color` | Color | integer |  | default computed dynamically (_get_default_color) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `sales_team` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `sales_team` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | no | no | no | `sales_team` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sales_team` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sales_team` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sales_team.sales_team_crm_tag_view_form` | form |  | `name`, `color` |  |  | `sales_team` |
| `sales_team.sales_team_crm_tag_view_tree` | list |  | `name`, `color` |  |  | `sales_team` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sales_team.sales_team_crm_tag_action` | Tags |  |  |  |  | `sales_team` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `crm.menu_crm_lead_categ` | Tags | `menu_crm_config_lead` | `sales_team.sales_team_crm_tag_action` | 1 |  |
| `sale.menu_tag_config` | Tags |  | `sales_team.sales_team_crm_tag_action` | 10 |  |

Machine-readable definition: `../../../schemas/data/entities/crm.tag.json`; views: `../../../schemas/interfaces/views/crm.tag.json`.
