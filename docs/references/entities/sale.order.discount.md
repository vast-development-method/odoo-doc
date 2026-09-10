# Discount Wizard (`sale.order.discount`)

**Transport name:** `sale.order.discount`  
**Storage name:** `sale_order_discount`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sale`

Description: Discount Wizard

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sale_order_id` | Sale Order | many to one | `sale.order` | required; default computed dynamically (lambda self: self.env.context.get('active_id')) |
| `company_id` | Company | many to one |  | related through path `sale_order_id.company_id` |
| `currency_id` | Currency | many to one |  | related through path `sale_order_id.currency_id` |
| `discount_amount` | Amount | monetary |  |  |
| `discount_percentage` | Percentage | float |  |  |
| `discount_type` | Discount Type | selection |  | default `sol_discount` |

## Selection values

### `discount_type` (Discount Type)

| Value | Label |
|---|---|
| `sol_discount` | On All Order Lines |
| `so_discount` | Global Discount |
| `amount` | Fixed Amount |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_discount_amount` | validation | self | `sale` | constrains: `discount_type`, `discount_percentage` |  |
| `_prepare_discount_product_values` | preparation rule | self | `sale` |  |  |
| `_prepare_global_discount_so_lines` | preparation rule | self, base_lines | `sale` |  |  |
| `_get_discount_product` | preparation rule | self | `sale` |  | Return product.product used for discount line |
| `_create_discount_lines` | internal rule | self | `sale` |  |  |
| `action_apply_discount` | user action | self | `sale` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_discount_amount` | ValidationError | Invalid discount amount | `sale` |
| `_get_discount_product` | ValidationError | There does not seem to be any discount product configured for this company yet. You can either use a per-line discount, or ask an administrator to grant the discount the first time. | `sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale.sale_order_line_wizard_form` | form |  | `sale_order_id`, `company_id`, `currency_id`, `discount_amount`, `discount_percentage`, `discount_type` | `Apply`, `Discard` |  | `sale` |

Machine-readable definition: `../../../schemas/data/entities/sale.order.discount.json`; views: `../../../schemas/interfaces/views/sale.order.discount.json`.
