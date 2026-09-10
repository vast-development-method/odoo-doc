# Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom) (`mrp.consumption.warning`)

**Transport name:** `mrp.consumption.warning`  
**Storage name:** `mrp_consumption_warning`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`

Description: Wizard in case of consumption in warning/strict and more component has been used for a MO (related to the bom)

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mrp_production_ids` | Manufacturing Production | many to many | `mrp.production` |  |
| `mrp_production_count` | Manufacturing Production Count | integer |  | computed by rule `_compute_mrp_production_count` (not stored) |
| `consumption` | Consumption | selection |  | computed by rule `_compute_consumption` (not stored) |
| `mrp_consumption_warning_line_ids` | Manufacturing Consumption Warning Line | one to many | `mrp.consumption.warning.line` | inverse field `mrp_consumption_warning_id` |

## Selection values

### `consumption` (Consumption)

| Value | Label |
|---|---|
| `flexible` | Allowed |
| `warning` | Allowed with warning |
| `strict` | Blocked |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_mrp_production_count` | computation | self | `mrp` | depends: `mrp_production_ids` |  |
| `_compute_consumption` | computation | self | `mrp` | depends: `mrp_consumption_warning_line_ids.consumption` |  |
| `action_confirm` | user action | self | `mrp` |  |  |
| `action_set_qty` | user action | self | `mrp` |  |  |
| `action_cancel` | user action | self | `mrp` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_set_qty` | UserError | Values cannot be set and validated because a Lot/Serial Number needs to be specified for a tracked product that is having its consumed amount increased:%(products)s | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `base.group_portal` | yes | yes | yes | no | `mrp_subcontracting` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| MRP Consumption Warnings Subcontractor | `[(4, ref('base.group_portal'))]` | `[('mrp_production_ids', 'in', user.partner_id.commercial_partner_id.production_ids.ids)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_mrp_consumption_warning_form` | form |  | `mrp_production_ids`, `consumption`, `mrp_production_count`, `mrp_consumption_warning_line_ids`, `mrp_production_id`, `consumption`, `product_id`, `product_uom_id`, `product_expected_qty_uom`, `product_consumed_qty_uom` | `Force`, `Confirm`, `Set Quantities & Validate`, `Discard` |  | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.action_mrp_consumption_warning` | Consumption Warning | form |  |  | new | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.consumption.warning.json`; views: `../../../schemas/interfaces/views/mrp.consumption.warning.json`.
