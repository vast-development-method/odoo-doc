# Warn Insufficient Repair Quantity (`stock.warn.insufficient.qty.repair`)

**Transport name:** `stock.warn.insufficient.qty.repair`  
**Storage name:** `stock_warn_insufficient_qty_repair`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `repair`

Description: Warn Insufficient Repair Quantity

## Identity and behavior

- Mixins (classical inheritance): `stock.warn.insufficient.qty`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `repair_id` | Repair | many to one | `repair.order` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_reference_document_company_id` | preparation rule | self | `repair` |  |  |
| `action_done` | user action | self | `repair` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `repair` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `repair.stock_warn_insufficient_qty_repair_form_view` | xpath | `stock.stock_warn_insufficient_qty_form_view` | `quantity`, `product_uom_name`, `location_id` |  |  | `repair` |

Machine-readable definition: `../../../schemas/data/entities/stock.warn.insufficient.qty.repair.json`; views: `../../../schemas/interfaces/views/stock.warn.insufficient.qty.repair.json`.
