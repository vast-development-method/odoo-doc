# Multiple order invoice creation (`pos.make.invoice`)

**Transport name:** `pos.make.invoice`  
**Storage name:** `pos_make_invoice`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `point_of_sale`

Description: Multiple order invoice creation

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `consolidated_billing` | Consolidated Billing | boolean |  | default `True`; Help: Create one invoice for all orders related to same customer and same invoicing address |
| `count` | Order Count | integer |  | computed by rule `_compute_order_count` (not stored) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_order_count` | computation | self | `point_of_sale` |  |  |
| `action_create_invoices` | user action | self | `point_of_sale` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_create_invoices` | UserError | No valid orders were selected. No new invoices could be generated | `point_of_sale` |
| `action_create_invoices` | UserError | The following refund orders can't be part of a consolidated invoice because they refunded invoiced orders. Each refund order should be handled separately.  %s | `point_of_sale` |
| `action_create_invoices` | UserError | Kindly ensure that each order contains a customer. | `point_of_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_user` | yes | yes | yes | no | `point_of_sale` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_make_invoice` | form |  | `count`, `consolidated_billing` | `Create`, `Cancel` |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.make.invoice.json`; views: `../../../schemas/interfaces/views/pos.make.invoice.json`.
