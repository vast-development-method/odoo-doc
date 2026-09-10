# Loyalty Communication (`loyalty.mail`)

**Transport name:** `loyalty.mail`  
**Storage name:** `loyalty_mail`  
**Kind:** persistent entity (one table)  
**Defined by package:** `loyalty`  
**Extended by packages:** `pos_loyalty`

Description: Loyalty Communication

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `program_id` | Program | many to one | `loyalty.program` | required; indexed; on delete of the target: cascade |
| `trigger` | When | selection |  | required |
| `points` | Points | float |  |  |
| `mail_template_id` | Email Template | many to one | `mail.template` | required; on delete of the target: cascade; restricted by domain `[["model", "=", "loyalty.card"]]` |
| `pos_report_print_id` | Print Report | many to one | `ir.actions.report` | restricted by domain `[["model", "=", "loyalty.card"]]`; Help: The report action to be executed when creating a coupon/gift card/loyalty card in the PoS. |

## Selection values

### `trigger` (When)

| Value | Label |
|---|---|
| `create` | At Creation |
| `points_reach` | When Reaching |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | no | no | no | `loyalty` |
| `point_of_sale.group_pos_user` | no | yes | no | no | `pos_loyalty` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_loyalty` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_loyalty` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_loyalty` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_mail_view_tree` | list |  | `trigger`, `points`, `mail_template_id` |  |  | `loyalty` |
| `pos_loyalty.loyalty_mail_view_tree_inherit_pos_loyalty` | field | `loyalty.loyalty_mail_view_tree` | `mail_template_id`, `pos_report_print_id` |  |  | `pos_loyalty` |

Machine-readable definition: `../../../schemas/data/entities/loyalty.mail.json`; views: `../../../schemas/interfaces/views/loyalty.mail.json`.
