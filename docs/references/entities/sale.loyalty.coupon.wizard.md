# Sale Loyalty - Apply Coupon Wizard (`sale.loyalty.coupon.wizard`)

**Transport name:** `sale.loyalty.coupon.wizard`  
**Storage name:** `sale_loyalty_coupon_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sale_loyalty`

Description: Sale Loyalty - Apply Coupon Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `order_id` | Order | many to one | `sale.order` | required; default computed dynamically (lambda self: self.env.context.get('active_id')) |
| `coupon_code` | Coupon Code | single line text |  | required |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_apply` | user action | self | `sale_loyalty` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_apply` | ValidationError | Invalid sales order. | `sale_loyalty` |
| `action_apply` | ValidationError | status['error'] | `sale_loyalty` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale_loyalty.sale_loyalty_coupon_wizard_view_form` | form |  | `coupon_code` | `Apply`, `Discard` |  | `sale_loyalty` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale_loyalty.sale_loyalty_coupon_wizard_action` | Enter Promotion or Coupon Code | form |  |  | new | `sale_loyalty` |

Machine-readable definition: `../../../schemas/data/entities/sale.loyalty.coupon.wizard.json`; views: `../../../schemas/interfaces/views/sale.loyalty.coupon.wizard.json`.
