# Point of Sale Details Report (`pos.details.wizard`)

**Transport name:** `pos.details.wizard`  
**Storage name:** `pos_details_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `point_of_sale`

Description: Point of Sale Details Report

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `start_date` | Start Date | date and time |  | required; default computed dynamically (_default_start_date) |
| `end_date` | End Date | date and time |  | required; default computed dynamically (fields.Datetime.now) |
| `pos_config_ids` | Point of sale Config | many to many | `pos.config` | default computed dynamically (lambda s: s.env['pos.config'].search([])); association table `pos_detail_configs` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_start_date` | preparation rule | self | `point_of_sale` |  | Find the earliest start_date of the latests sessions |
| `_onchange_start_date` | on change | self | `point_of_sale` | onchange: `start_date` |  |
| `_onchange_end_date` | on change | self | `point_of_sale` | onchange: `end_date` |  |
| `generate_report` | operation | self | `point_of_sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_manager` | yes | yes | yes | no | `point_of_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_details_wizard` | form |  | `start_date`, `end_date`, `pos_config_ids` | `Print`, `Cancel` |  | `point_of_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_report_pos_details` | Sales Details | form |  |  | new | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.details.wizard.json`; views: `../../../schemas/interfaces/views/pos.details.wizard.json`.
