# Point of Sale Printer (`pos.printer`)

**Transport name:** `pos.printer`  
**Storage name:** `pos_printer`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`

Description: Point of Sale Printer

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Printer Name | single line text |  | required; default `Printer`; Help: An internal identification of the printer |
| `printer_type` | Printer Type | selection |  | default `iot` |
| `proxy_ip` | Proxy internet protocol Address | single line text |  | Help: The IP Address or hostname of the Printer's hardware proxy |
| `product_categories_ids` | Printed Product Categories | many to many | `pos.category` | association table `printer_category_rel` |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `pos_config_ids` | Point of sale Config | many to many | `pos.config` | association table `pos_config_printer_rel` |
| `epson_printer_ip` | Epson Printer internet protocol Address | single line text |  | default `0.0.0.0`; Help: Local IP address of an Epson receipt printer, or its serial number if the 'Automatic Certificate Update' option is enabled in the printer settings. |

## Selection values

### `printer_type` (Printer Type)

| Value | Label |
|---|---|
| `iot` | Use a printer connected to the IoT Box |
| `epson_epos` | Use an Epson printer |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `use_local_network_access` | operation | self | `point_of_sale` | model |  |
| `_constrains_epson_printer_ip` | validation | self | `point_of_sale` | constrains: `epson_printer_ip` |  |
| `_onchange_epson_printer_ip` | on change | self | `point_of_sale` | onchange: `epson_printer_ip` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_epson_printer_ip` | ValidationError | Epson Printer IP Address cannot be empty. | `point_of_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `point_of_sale.group_pos_user` | no | yes | no | no | `point_of_sale` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_printer_form` | form |  | `company_id`, `name`, `printer_type`, `epson_printer_ip`, `proxy_ip`, `product_categories_ids` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_printer` | list |  | `name`, `product_categories_ids`, `proxy_ip`, `company_id` |  |  | `point_of_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_printer_form` | Preparation Printers | list,kanban,form |  |  |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.printer.json`; views: `../../../schemas/interfaces/views/pos.printer.json`.
