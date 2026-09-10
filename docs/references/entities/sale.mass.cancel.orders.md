# Cancel multiple quotations (`sale.mass.cancel.orders`)

**Transport name:** `sale.mass.cancel.orders`  
**Storage name:** `sale_mass_cancel_orders`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sale`

Description: Cancel multiple quotations

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sale_order_ids` | Sale orders to cancel | many to many | `sale.order` | default computed dynamically (lambda self: self.env.context.get('active_ids')); association table `sale_order_mass_cancel_wizard_rel` |
| `sale_orders_count` | Sale Orders Count | integer |  | computed by rule `_compute_sale_orders_count` (not stored) |
| `has_confirmed_order` | Has Confirmed Order | boolean |  | computed by rule `_compute_has_confirmed_order` (not stored) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_sale_orders_count` | computation | self | `sale` | depends: `sale_order_ids` |  |
| `_compute_has_confirmed_order` | computation | self | `sale` | depends: `sale_order_ids` |  |
| `action_mass_cancel` | user action | self | `sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Sales Mass Cancel Orders: access only your own wizard | global (all users) | `[('create_uid', '=', user.id)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale.mass_cancel_orders_view_form` | form |  | `sale_order_ids`, `sale_orders_count`, `has_confirmed_order`, `sale_orders_count` | `Cancel`, `Discard` |  | `sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale.action_mass_cancel_orders` | Cancel | form |  |  | new | `sale` |

Machine-readable definition: `../../../schemas/data/entities/sale.mass.cancel.orders.json`; views: `../../../schemas/interfaces/views/sale.mass.cancel.orders.json`.
