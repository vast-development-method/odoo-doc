# customer relationship management Recurring revenue plans (`crm.recurring.plan`)

**Transport name:** `crm.recurring.plan`  
**Storage name:** `crm_recurring_plan`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm`

Description: CRM Recurring revenue plans

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Plan Name | single line text |  | required; translatable |
| `number_of_months` | # Months | integer |  | required |
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | default `10` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_number_of_months` | Constraint | `CHECK(number_of_months >= 0)` | The number of month can't be negative. | `crm` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.crm_recurring_plan_view_tree` | list |  | `sequence`, `name`, `number_of_months` |  |  | `crm` |
| `crm.crm_recurring_plan_view_search` | search |  | `name` |  | `Archived` | `crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.crm_recurring_plan_action` | Recurring Plans | list |  |  |  | `crm` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `crm.crm_recurring_plan_menu_config` | Recurring Plans | `crm_menu_config` | `crm.crm_recurring_plan_action` | 12 | `crm.group_use_recurring_revenues` |

Machine-readable definition: `../../../schemas/data/entities/crm.recurring.plan.json`; views: `../../../schemas/interfaces/views/crm.recurring.plan.json`.
