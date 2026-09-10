# Warn Insufficient Scrap Quantity (`stock.warn.insufficient.qty.scrap`)

**Transport name:** `stock.warn.insufficient.qty.scrap`  
**Storage name:** `stock_warn_insufficient_qty_scrap`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Warn Insufficient Scrap Quantity

## Identity and behavior

- Mixins (classical inheritance): `stock.warn.insufficient.qty`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `scrap_id` | Scrap | many to one | `stock.scrap` |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_reference_document_company_id` | preparation rule | self | `stock` |  |  |
| `action_done` | user action | self | `stock` |  |  |
| `action_cancel` | user action | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_warn_insufficient_qty_scrap_form_view` | xpath | `stock.stock_warn_insufficient_qty_form_view` | `quantity`, `product_uom_name`, `location_id` |  |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.warn.insufficient.qty.scrap.json`; views: `../../../schemas/interfaces/views/stock.warn.insufficient.qty.scrap.json`.
