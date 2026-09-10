# History for Loyalty cards and Ewallets (`loyalty.history`)

**Transport name:** `loyalty.history`  
**Storage name:** `loyalty_history`  
**Kind:** persistent entity (one table)  
**Defined by package:** `loyalty`  
**Extended by packages:** `sale_loyalty`

Description: History for Loyalty cards and Ewallets

## Identity and behavior

- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `card_id` | Card | many to one | `loyalty.card` | required; indexed; on delete of the target: cascade |
| `company_id` | Company | many to one |  | related through path `card_id.company_id` |
| `description` | Description | multi line text |  | required |
| `issued` | Issued | float |  |  |
| `used` | Used | float |  |  |
| `order_model` | Order Model | single line text |  | read only |
| `order_id` | Order | many to one by reference |  | read only |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_order_portal_url` | preparation rule | self | `loyalty`, `sale_loyalty` |  |  |
| `_get_order_description` | preparation rule | self | `loyalty` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | no | no | no | `loyalty` |
| `point_of_sale.group_pos_user` | yes | yes | yes | no | `pos_loyalty` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Loyalty history multi company rule | global (all users) | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_history_form` | form |  | `card_id`, `description`, `order_id`, `order_model`, `issued`, `used` |  |  | `loyalty` |

Machine-readable definition: `../../../schemas/data/entities/loyalty.history.json`; views: `../../../schemas/interfaces/views/loyalty.history.json`.
