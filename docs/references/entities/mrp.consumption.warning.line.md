# Line of issue consumption (`mrp.consumption.warning.line`)

**Transport name:** `mrp.consumption.warning.line`  
**Storage name:** `mrp_consumption_warning_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`

Description: Line of issue consumption

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mrp_consumption_warning_id` | Parent Wizard | many to one | `mrp.consumption.warning` | required; read only; on delete of the target: cascade |
| `mrp_production_id` | Manufacturing Order | many to one | `mrp.production` | required; read only; on delete of the target: cascade |
| `consumption` | Consumption | selection |  | related through path `mrp_production_id.consumption` |
| `product_id` | Product | many to one | `product.product` | required; read only |
| `product_uom_id` | Unit | many to one | `uom.uom` | read only; related through path `product_id.uom_id` |
| `product_consumed_qty_uom` | Consumed | float |  | read only |
| `product_expected_qty_uom` | To Consume | float |  | read only |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `base.group_portal` | yes | yes | yes | no | `mrp_subcontracting` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| MRP Consumption Warning Lines Subcontractor | `[(4, ref('base.group_portal'))]` | `[('mrp_production_id', 'in', user.partner_id.commercial_partner_id.production_ids.ids)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/mrp.consumption.warning.line.json`.
