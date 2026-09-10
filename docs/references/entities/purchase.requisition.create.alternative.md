# Wizard to preset values for alternative purchase order (`purchase.requisition.create.alternative`)

**Transport name:** `purchase.requisition.create.alternative`  
**Storage name:** `purchase_requisition_create_alternative`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `purchase_requisition`  
**Extended by packages:** `purchase_requisition_sale`, `purchase_requisition_stock`

Description: Wizard to preset values for alternative PO

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `origin_po_id` | Origin Purchase order | many to one | `purchase.order` | Help: The original PO that this alternative PO is being created for. |
| `partner_ids` | Vendor | many to many | `res.partner` | required; Help: Choose a vendor for alternative PO |
| `purchase_warn_msg` | Warning Messages | multi line text |  | computed by rule `_compute_purchase_warn_msg` (not stored); visible only to groups `purchase.group_warning_purchase` |
| `copy_products` | Copy Products | boolean |  | default `True`; Help: If this is checked, the product quantities of the original PO will be copied |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_purchase_warn_msg` | computation | self | `purchase_requisition` | depends: `partner_ids`, `copy_products` |  |
| `action_create_alternative` | user action | self | `purchase_requisition` |  |  |
| `_get_alternative_values` | preparation rule | self | `purchase_requisition_stock`, `purchase_requisition` |  |  |
| `_get_alternative_line_value` | preparation rule | self, order_line, product_tmpl_ids_with_description | `purchase_requisition_sale`, `purchase_requisition_stock`, `purchase_requisition` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase_requisition.purchase_requisition_create_alternative_form` | form |  | `origin_po_id`, `partner_ids`, `copy_products`, `purchase_warn_msg` | `Create Alternative`, `Cancel` |  | `purchase_requisition` |

Machine-readable definition: `../../../schemas/data/entities/purchase.requisition.create.alternative.json`; views: `../../../schemas/interfaces/views/purchase.requisition.create.alternative.json`.
