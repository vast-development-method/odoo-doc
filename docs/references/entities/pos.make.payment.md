# Point of Sale Make Payment Wizard (`pos.make.payment`)

**Transport name:** `pos.make.payment`  
**Storage name:** `pos_make_payment`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `point_of_sale`

Description: Point of Sale Make Payment Wizard

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `config_id` | Point of Sale Configuration | many to one | `pos.config` | required; default computed dynamically (_default_config) |
| `amount` | Amount | float |  | required; default computed dynamically (_default_amount) |
| `payment_method_id` | Payment Method | many to one | `pos.payment.method` | required; default computed dynamically (_default_payment_method) |
| `payment_name` | Payment Reference | single line text |  |  |
| `payment_date` | Payment Date | date and time |  | required; default computed dynamically (lambda self: fields.Datetime.now()) |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_config` | preparation rule | self | `point_of_sale` |  |  |
| `_default_amount` | preparation rule | self | `point_of_sale` |  |  |
| `_default_payment_method` | preparation rule | self | `point_of_sale` |  |  |
| `check` | operation | self | `point_of_sale` |  | Check the order: if the order is not paid: continue payment, if the order is paid print ticket. |
| `launch_payment` | operation | self | `point_of_sale` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `check` | UserError | Customer is required for %s payment method. | `point_of_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_manager` | yes | yes | yes | no | `point_of_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_payment` | form |  | `config_id`, `payment_method_id`, `amount`, `payment_name` | `Make Payment`, `Cancel` |  | `point_of_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_payment` | Payment | form |  |  | new | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.make.payment.json`; views: `../../../schemas/interfaces/views/pos.make.payment.json`.
