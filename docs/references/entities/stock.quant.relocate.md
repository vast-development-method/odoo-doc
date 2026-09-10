# Stock Quantity Relocation (`stock.quant.relocate`)

**Transport name:** `stock.quant.relocate`  
**Storage name:** `stock_quant_relocate`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Stock Quantity Relocation

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `quant_ids` | Quant | many to many | `stock.quant` |  |
| `company_id` | Company | many to one |  | related through path `quant_ids.company_id` |
| `dest_location_id` | Dest Location | many to one | `stock.location` | restricted by domain `[('usage', '=', 'internal'), ('company_id', '=', company_id)]` |
| `dest_package_id_domain` | Dest Package Identifier Domain | single line text |  | computed by rule `_compute_dest_package_id_domain` (not stored) |
| `dest_package_id` | Dest Package | many to one | `stock.package` | computed by rule `_compute_dest_package_id` and stored; restricted by domain `dest_package_id_domain` |
| `message` | Reason for relocation | multi line text |  |  |
| `is_partial_package` | Is Partial Package | boolean |  | computed by rule `_compute_is_partial_package` (not stored) |
| `partial_package_names` | Partial Package Names | single line text |  | computed by rule `_compute_is_partial_package` (not stored) |
| `is_multi_location` | Is Multi Location | boolean |  | computed by rule `_compute_is_multi_location` (not stored) |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_partial_package` | computation | self | `stock` | depends: `quant_ids` |  |
| `_compute_is_multi_location` | computation | self | `stock` | depends: `dest_location_id`, `quant_ids` |  |
| `_compute_dest_package_id_domain` | computation | self | `stock` | depends: `dest_location_id`, `quant_ids` |  |
| `_compute_dest_package_id` | computation | self | `stock` | depends: `dest_package_id_domain` |  |
| `action_relocate_quants` | user action | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_quant_relocate_view_form` | form |  | `quant_ids`, `company_id`, `dest_location_id`, `dest_package_id_domain`, `dest_package_id`, `message`, `is_partial_package`, `is_multi_location`, `partial_package_names` | `Confirm`, `Discard` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.quant.relocate.json`; views: `../../../schemas/interfaces/views/stock.quant.relocate.json`.
