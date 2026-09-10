# Update Loyalty Card Points (`loyalty.card.update.balance`)

**Transport name:** `loyalty.card.update.balance`  
**Storage name:** `loyalty_card_update_balance`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `loyalty`

Description: Update Loyalty Card Points

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `card_id` | Card | many to one | `loyalty.card` | required; read only |
| `old_balance` | Old Balance | float |  | related through path `card_id.points` |
| `new_balance` | New Balance | float |  |  |
| `description` | Description | single line text |  | required |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_update_card_point` | user action | self | `loyalty` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_update_card_point` | ValidationError | New Balance should be positive and different then old balance. | `loyalty` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | no | no | no | `loyalty` |
| `point_of_sale.group_pos_user` | yes | yes | yes | no | `pos_loyalty` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_card_update_balance_form` | form |  | `card_id`, `old_balance`, `new_balance`, `description` | `action_update_card_point`,  |  | `loyalty` |

Machine-readable definition: `../../../schemas/data/entities/loyalty.card.update.balance.json`; views: `../../../schemas/interfaces/views/loyalty.card.update.balance.json`.
