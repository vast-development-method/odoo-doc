# Purchase Line and Vendor Bill line matching view (`purchase.bill.line.match`)

**Transport name:** `purchase.bill.line.match`  
**Storage name:** `purchase_bill_line_match`  
**Kind:** persistent entity (one table)  
**Defined by package:** `purchase`

Description: Purchase Line and Vendor Bill line matching view

## Identity and behavior

- Default ordering: `product_id, aml_id, pol_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `pol_id` | Pol | many to one | `purchase.order.line` | read only |
| `aml_id` | Aml | many to one | `account.move.line` | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `partner_id` | Partner | many to one | `res.partner` | read only |
| `product_id` | Product | many to one | `product.product` | read only |
| `line_qty` | Line Qty | float |  | read only |
| `line_uom_id` | Line Unit of measure | many to one | `uom.uom` | read only |
| `qty_invoiced` | Qty Invoiced | float |  | read only |
| `qty_to_invoice` | Qty to invoice | float |  | read only |
| `purchase_order_id` | Purchase Order | many to one | `purchase.order` | read only |
| `account_move_id` | Account Move | many to one | `account.move` | read only |
| `line_amount_untaxed` | Line Amount Untaxed | monetary |  | read only |
| `currency_id` | Currency | many to one | `res.currency` | read only |
| `state` | State | single line text |  | read only |
| `product_uom_id` | Product Unit of measure | many to one | `uom.uom` | related through path `product_id.uom_id` |
| `product_uom_qty` | Product Unit of measure Qty | float |  | computed by rule `_compute_product_uom_qty` (not stored); writable through an inverse rule |
| `product_uom_price` | Product Unit of measure Price | float |  | computed by rule `_compute_product_uom_price` (not stored); writable through an inverse rule |
| `billed_amount_untaxed` | Billed Amount Untaxed | monetary |  | computed by rule `_compute_amount_untaxed_fields` (not stored); currency taken from `currency_id` |
| `purchase_amount_untaxed` | Purchase Amount Untaxed | monetary |  | computed by rule `_compute_amount_untaxed_fields` (not stored); currency taken from `currency_id` |
| `reference` | Reference | single line text |  | computed by rule `_compute_reference` (not stored) |

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_inverse_product_uom_price` | on change | self | `purchase` | onchange: `product_uom_price` |  |
| `_inverse_product_uom_qty` | on change | self | `purchase` | onchange: `product_uom_qty` |  |
| `_compute_amount_untaxed_fields` | computation | self | `purchase` |  |  |
| `_compute_reference` | computation | self | `purchase` |  |  |
| `_compute_display_name` | computation | self | `purchase` |  |  |
| `_compute_product_uom_qty` | computation | self | `purchase` |  |  |
| `_compute_product_uom_price` | computation | self | `purchase` | depends: `aml_id.price_unit`, `pol_id.price_unit` |  |
| `_select_po_line` | internal rule | self | `purchase` | model |  |
| `_select_am_line` | internal rule | self | `purchase` | model |  |
| `_table_query` | internal rule | self | `purchase` |  |  |
| `action_open_line` | user action | self | `purchase` |  |  |
| `_action_create_bill_from_po_lines` | internal rule | self, partner, po_lines | `purchase` | model | Create a new vendor bill with the selected PO lines and returns an action to open it |
| `action_match_lines` | user action | self | `purchase` |  |  |
| `action_add_to_po` | user action | self | `purchase` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_match_lines` | UserError | You must select at least one Purchase Order line to match or create bill. | `purchase` |
| `action_add_to_po` | UserError | Select Vendor Bill lines to add to a Purchase Order | `purchase` |
| `action_add_to_po` | UserError | Please select bill lines with the same vendor. | `purchase` |
| `action_add_to_po` | UserError | Vendor Bill lines can only be added to one Purchase Order. | `purchase` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_purchase_user` | no | yes | no | no | `purchase` |
| `account.group_account_readonly` | no | yes | no | no | `purchase` |
| `account.group_account_invoice` | no | yes | yes | no | `purchase` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase.purchase_bill_line_match_tree` | list |  | `partner_id`, `reference`, `display_name`, `product_uom_qty`, `qty_invoiced`, `qty_to_invoice`, `product_uom_id`, `product_uom_price`, `billed_amount_untaxed`, `purchase_amount_untaxed`, `currency_id` | `Match`, `Add to PO` |  | `purchase` |

Machine-readable definition: `../../../schemas/data/entities/purchase.bill.line.match.json`; views: `../../../schemas/interfaces/views/purchase.bill.line.match.json`.
