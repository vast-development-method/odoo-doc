# Assign serial numbers to production order (`mrp.production.serials`)

**Transport name:** `mrp.production.serials`  
**Storage name:** `mrp_production_serials`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`  
**Extended by packages:** `mrp_subcontracting`

Description: Assign serial numbers to production order

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `production_id` | Production | many to one | `mrp.production` |  |
| `workorder_id` | Workorder | many to one | `mrp.workorder` |  |
| `lot_name` | First SN | single line text |  | computed by rule `_compute_lot_name` and stored |
| `lot_quantity` | Number of SN | integer |  | computed by rule `_compute_lot_quantity` and stored |
| `serial_numbers` | Produced Serial Numbers | multi line text |  | computed by rule `_compute_lot_name` and stored |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_lot_name` | computation | self | `mrp` | depends: `production_id` |  |
| `_compute_lot_quantity` | computation | self | `mrp` | depends: `production_id` |  |
| `_onchange_serial_numbers` | on change | self | `mrp` | onchange: `serial_numbers` |  |
| `action_generate_serial_numbers` | user action | self | `mrp` |  |  |
| `action_split_and_assign_serials` | user action | self | `mrp` |  |  |
| `action_apply` | user action | self | `mrp_subcontracting`, `mrp` |  |  |
| `_closing_action` | internal rule | self, mos | `mrp` |  |  |
| `_parse_serial_numbers` | internal rule | self | `mrp` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_parse_serial_numbers` | UserError | There is no serial numbers to apply. | `mrp` |
| `_parse_serial_numbers` | UserError | No valid serial numbers provided. | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `base.group_portal` | yes | yes | yes | no | `mrp_subcontracting` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_mrp_production_serials_form` | form |  | `production_id`, `lot_name`, `lot_quantity`, `serial_numbers` | `Generate`, `Apply`, `Prepare MO`, `Discard` |  | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.action_assign_serial_numbers` | Assign Serial Numbers | form |  | `{}` | new | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.production.serials.json`; views: `../../../schemas/interfaces/views/mrp.production.serials.json`.
