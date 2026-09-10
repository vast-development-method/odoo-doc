# Sale Loyalty - Reward Selection Wizard (`sale.loyalty.reward.wizard`)

**Transport name:** `sale.loyalty.reward.wizard`  
**Storage name:** `sale_loyalty_reward_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sale_loyalty`

Description: Sale Loyalty - Reward Selection Wizard

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `order_id` | Order | many to one | `sale.order` | required; default computed dynamically (lambda self: self.env.context.get('active_id')) |
| `reward_ids` | Reward | many to many | `loyalty.reward` | computed by rule `_compute_claimable_reward_ids` (not stored) |
| `selected_reward_id` | Selected Reward | many to one | `loyalty.reward` | restricted by domain `[('id', 'in', reward_ids)]` |
| `multi_product_reward` | Multi Product Reward | boolean |  | related through path `selected_reward_id.multi_product` |
| `reward_product_ids` | Reward Product | many to many |  | related through path `selected_reward_id.reward_product_ids` |
| `selected_product_id` | Selected Product | many to one | `product.product` | computed by rule `_compute_selected_product_id` and stored; restricted by domain `[('id', 'in', reward_product_ids)]` |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_claimable_reward_ids` | computation | self | `sale_loyalty` | depends: `order_id` |  |
| `_compute_selected_product_id` | computation | self | `sale_loyalty` | depends: `reward_product_ids` |  |
| `action_apply` | user action | self | `sale_loyalty` |  |  |
| `action_cancel` | user action | self | `sale_loyalty` |  |  |
| `_unlink_unused_coupon_ids` | internal rule | self | `sale_loyalty` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_apply` | ValidationError | No reward selected. | `sale_loyalty` |
| `action_apply` | ValidationError | Coupon not found while trying to add the following reward: %s | `sale_loyalty` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale_loyalty.sale_loyalty_reward_wizard_view_form` | form |  | `order_id`, `reward_ids`, `multi_product_reward`, `reward_product_ids`, `selected_reward_id`, `selected_product_id` | `Apply`, `Discard`, `Discard`, `Coupons & Loyalty` |  | `sale_loyalty` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale_loyalty.sale_loyalty_reward_wizard_action` | Available Rewards | form |  |  | new | `sale_loyalty` |

Machine-readable definition: `../../../schemas/data/entities/sale.loyalty.reward.wizard.json`; views: `../../../schemas/interfaces/views/sale.loyalty.reward.wizard.json`.
