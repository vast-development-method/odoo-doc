# Bill to Purchase Order (`bill.to.po.wizard`)

**Transport name:** `bill.to.po.wizard`  
**Storage name:** `bill_to_po_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `purchase`

Description: Bill to Purchase Order

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `purchase_order_id` | Purchase Order | many to one | `purchase.order` |  |
| `partner_id` | Partner | many to one | `res.partner` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_add_to_po` | user action | self | `purchase` |  |  |
| `action_add_downpayment` | user action | self | `purchase` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_add_to_po` | UserError | There are no products to add to the Purchase Order. Are these Down Payments? | `purchase` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_purchase_user` | yes | yes | yes | no | `purchase` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase.bill_to_po_wizard_form` | form |  | `purchase_order_id` | `Add Products`, `Add Down Payment` |  | `purchase` |

Machine-readable definition: `../../../schemas/data/entities/bill.to.po.wizard.json`; views: `../../../schemas/interfaces/views/bill.to.po.wizard.json`.
