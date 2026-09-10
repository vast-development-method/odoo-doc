# Scrap (`stock.scrap`)

**Transport name:** `stock.scrap`  
**Storage name:** `stock_scrap`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `mrp`

Description: Scrap

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reference | single line text |  | required; read only; default computed dynamically (lambda self: _('New')); not copied on duplication |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `origin` | Source Document | single line text |  |  |
| `product_id` | Product | many to one | `product.product` | required; restricted by domain `[('type', '=', 'consu')]`; must belong to the same company |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `product_uom_id` | Unit | many to one | `uom.uom` | required; computed by rule `_compute_product_uom_id` and stored; restricted by domain `[('id', 'in', allowed_uom_ids)]`; precomputed before insertion |
| `tracking` | Product Tracking | selection |  | read only; related through path `product_id.tracking` |
| `lot_id` | Lot/Serial | many to one | `stock.lot` | restricted by domain `[('product_id', '=', product_id)]`; must belong to the same company |
| `package_id` | Package | many to one | `stock.package` | must belong to the same company |
| `owner_id` | Owner | many to one | `res.partner` | must belong to the same company |
| `move_ids` | Move | one to many | `stock.move` | inverse field `scrap_id` |
| `picking_id` | Picking | many to one | `stock.picking` | must belong to the same company |
| `location_id` | Source Location | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; restricted by domain `[('usage', '=', 'internal')]`; must belong to the same company; precomputed before insertion |
| `scrap_location_id` | Scrap Location | many to one | `stock.location` | required; computed by rule `_compute_scrap_location_id` and stored; restricted by domain `[('usage', '=', 'inventory')]`; must belong to the same company; precomputed before insertion |
| `scrap_qty` | Quantity | float |  | required; computed by rule `_compute_scrap_qty` and stored; default `1.0`; precision `Product Unit` |
| `state` | Status | selection |  | read only; default `draft`; changes are tracked in the message thread |
| `date_done` | Date | date and time |  | read only |
| `should_replenish` | Replenish Quantities | boolean |  | Help: Trigger replenishment for scrapped products |
| `scrap_reason_tag_ids` | Scrap Reason | many to many | `stock.scrap.reason.tag` |  |
| `production_id` | Manufacturing Order | many to one | `mrp.production` | indexed (btree_not_null); must belong to the same company |
| `workorder_id` | Work Order | many to one | `mrp.workorder` | indexed (btree_not_null); must belong to the same company |
| `product_is_kit` | Product Is Kit | boolean |  | related through path `product_id.is_kits` |
| `product_template` | Product Template | many to one |  | related through path `product_id.product_tmpl_id` |
| `bom_id` | Kit | many to one | `mrp.bom` | restricted by domain `[('type', '=', 'phantom'), '\|', ('product_id', '=', product_id), '&', ('product_id', '=', False), ('product_tmpl_id', '=', product_template)]`; must belong to the same company |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `done` | Done |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_allowed_uom_ids` | computation | self | `stock` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` |  |
| `_compute_product_uom_id` | computation | self | `stock` | depends: `product_id` |  |
| `_compute_location_id` | computation | self | `mrp`, `stock` | depends: `company_id`, `picking_id`; depends: `workorder_id`, `production_id` |  |
| `_compute_scrap_location_id` | computation | self | `stock` | depends: `company_id` |  |
| `_compute_scrap_qty` | computation | self | `mrp`, `stock` | depends: `move_ids`, `move_ids.move_line_ids.quantity`, `product_id` |  |
| `_onchange_serial_number` | on change | self | `mrp`, `stock` | onchange: `lot_id` |  |
| `_unlink_except_done` | internal rule | self | `stock` | ondelete |  |
| `_prepare_move_values` | preparation rule | self | `mrp`, `stock` |  |  |
| `do_scrap` | user action | self | `mrp`, `stock` |  |  |
| `_create_scrap_move` | internal rule | self | `mrp`, `stock` |  |  |
| `do_replenish` | user action | self, values | `mrp`, `stock` |  |  |
| `action_get_stock_picking` | user action | self | `stock` |  |  |
| `action_get_stock_move_lines` | user action | self | `stock` |  |  |
| `_should_check_available_qty` | internal rule | self | `mrp`, `stock` |  |  |
| `check_available_qty` | operation | self | `stock` |  |  |
| `action_validate` | user action | self | `stock` |  |  |
| `_onchange_product_id` | on change | self | `mrp` | onchange: `product_id` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_done` | UserError | You cannot delete a scrap which is done. | `stock` |
| `action_validate` | UserError | You can only enter positive quantities. | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| stock_scrap_company multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.stock_scrap_view_form2_mrp_inherit_mrp` | field | `stock.stock_scrap_form_view2` | `owner_id`, `workorder_id`, `production_id`, `product_is_kit`, `product_template`, `bom_id` |  |  | `mrp` |
| `mrp.stock_scrap_view_form_mrp_inherit_mrp` | field | `stock.stock_scrap_form_view` | `owner_id`, `workorder_id`, `production_id`, `product_is_kit`, `product_template`, `bom_id` |  |  | `mrp` |
| `mrp.stock_scrap_search_view_inherit_mrp` | xpath | `stock.stock_scrap_search_view` |  |  | `Draft`, `Done` | `mrp` |
| `stock.stock_scrap_search_view` | search |  | `name`, `product_id`, `location_id`, `scrap_location_id`, `create_date` |  | `Product`, `Location`, `Scrap Location`, `Transfer` | `stock` |
| `stock.stock_scrap_form_view` | form |  | `state`, `picking_id`, `move_ids`, `name`, `product_id`, `tracking`, `scrap_qty`, `product_uom_id`, `lot_id`, `should_replenish`, `scrap_reason_tag_ids`, `company_id`, `package_id`, `owner_id`, `location_id`, `scrap_location_id`, `origin`, `date_done`, `picking_id`, `company_id` | `Validate`, `Stock Operation`, `action_get_stock_move_lines` |  | `stock` |
| `stock.stock_scrap_view_kanban` | kanban |  | `name`, `date_done`, `product_id`, `scrap_qty`, `state` |  |  | `stock` |
| `stock.stock_scrap_tree_view` | list |  | `company_id`, `name`, `date_done`, `product_id`, `scrap_qty`, `product_uom_id`, `location_id`, `scrap_location_id`, `company_id`, `state` |  |  | `stock` |
| `stock.stock_scrap_form_view2` | form |  | `state`, `product_id`, `tracking`, `scrap_qty`, `product_uom_id`, `lot_id`, `scrap_reason_tag_ids`, `picking_id`, `package_id`, `owner_id`, `company_id`, `location_id`, `scrap_location_id`, `should_replenish` | `Scrap Products`, `Discard` |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_stock_scrap` | Scrap Orders | list,form,kanban,pivot,graph |  |  |  | `stock` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mrp.menu_mrp_scrap` | Scrap | `menu_mrp_manufacturing` | `stock.action_stock_scrap` | 25 |  |

Machine-readable definition: `../../../schemas/data/entities/stock.scrap.json`; views: `../../../schemas/interfaces/views/stock.scrap.json`.
