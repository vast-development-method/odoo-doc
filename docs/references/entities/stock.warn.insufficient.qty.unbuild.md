# Warn Insufficient Unbuild Quantity (`stock.warn.insufficient.qty.unbuild`)

**Transport name:** `stock.warn.insufficient.qty.unbuild`  
**Storage name:** `stock_warn_insufficient_qty_unbuild`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`

Description: Warn Insufficient Unbuild Quantity

## Identity and behavior

- Mixins (classical inheritance): `stock.warn.insufficient.qty`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `unbuild_id` | Unbuild | many to one | `mrp.unbuild` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_reference_document_company_id` | preparation rule | self | `mrp` |  |  |
| `action_done` | user action | self | `mrp` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.stock_warn_insufficient_qty_unbuild_form_view` | xpath | `stock.stock_warn_insufficient_qty_form_view` | `quantity`, `product_uom_name`, `location_id` |  |  | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/stock.warn.insufficient.qty.unbuild.json`; views: `../../../schemas/interfaces/views/stock.warn.insufficient.qty.unbuild.json`.
