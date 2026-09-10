# Stock Package History (`stock.package.history`)

**Transport name:** `stock.package.history`  
**Storage name:** `stock_package_history`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`

Description: Stock Package History

## Identity and behavior

- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `location_id` | Origin Location | many to one | `stock.location` |  |
| `location_dest_id` | Destination Location | many to one | `stock.location` |  |
| `move_line_ids` | Move Lines | one to many | `stock.move.line` | required; inverse field `package_history_id` |
| `package_id` | Package | many to one | `stock.package` | required; on delete of the target: cascade |
| `package_name` | Package Name | single line text |  | required |
| `package_type_id` | Package Type | many to one | `stock.package.type` | related through path `package_id.package_type_id` |
| `parent_orig_id` | Origin Container | many to one | `stock.package` |  |
| `parent_orig_name` | Origin Container Name | single line text |  |  |
| `parent_dest_id` | Destination Container | many to one | `stock.package` |  |
| `parent_dest_name` | Destination Container Name | single line text |  |  |
| `outermost_dest_id` | Outermost Destination Container | many to one | `stock.package` |  |
| `picking_ids` | Transfers | many to many | `stock.picking` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_complete_dest_name_except_outermost` | preparation rule | self | `stock` |  |  |
| `action_show_package` | user action | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.package_history_search_view` | search |  | `package_name`, `location_id`, `package_type_id` |  | `In internal locations`, `Main Packages`, `Location`, `Package Type` | `stock` |
| `stock.view_stock_package_history_list` | list |  | `package_name`, `package_type_id`, `location_id`, `parent_orig_id`, `location_dest_id`, `parent_dest_id`, `company_id` | `View` |  | `stock` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `stock.action_report_package_history_barcode` | Package Barcode with Contents | qweb-pdf | `stock.report_package_history_barcode` |  |  |
| `stock.action_report_package_history_barcode_small` | Package Barcode (PDF) | qweb-pdf | `stock.report_package_history_barcode_small` |  |  |
| `stock.label_package_history_template` | Package Barcode (ZPL) | qweb-text | `stock.label_package_history_template_view` |  |  |

Machine-readable definition: `../../../schemas/data/entities/stock.package.history.json`; views: `../../../schemas/interfaces/views/stock.package.history.json`.
