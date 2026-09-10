# Unbuild Order (`mrp.unbuild`)

**Transport name:** `mrp.unbuild`  
**Storage name:** `mrp_unbuild`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`

Description: Unbuild Order

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (16)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reference | single line text |  | read only; default computed dynamically (lambda s: s.env._('New')); not copied on duplication |
| `product_id` | Product | many to one | `product.product` | required; computed by rule `_compute_product_id` and stored; restricted by domain `[('type', '=', 'consu')]`; must belong to the same company; precomputed before insertion |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda s: s.env.company); indexed |
| `product_qty` | Quantity | float |  | required; computed by rule `_compute_product_qty` and stored; default `1.0`; precision `Product Unit`; precomputed before insertion |
| `product_uom_id` | Unit | many to one | `uom.uom` | required; computed by rule `_compute_product_uom_id` and stored; precomputed before insertion |
| `bom_id` | Bill of Material | many to one | `mrp.bom` | computed by rule `_compute_bom_id` and stored; restricted by domain `[         '\|',             ('product_id', '=', product_id),             '&',                 ('product_tmpl_id.product_variant_ids', '=', product_id),                 ('product_id','=',False),         ('type', '=', 'normal'),         '\|',             ('company_id', '=', company_id),             ('company_id', '=', False)         ]`; must belong to the same company |
| `mo_id` | Manufacturing Order | many to one | `mrp.production` | indexed (btree_not_null); restricted by domain `[('state', '=', 'done'), ('product_id', '=?', product_id), ('bom_id', '=?', bom_id)]`; must belong to the same company |
| `mo_bom_id` | Bill of Material used on the Production Order | many to one | `mrp.bom` | related through path `mo_id.bom_id` |
| `lot_producing_ids` | Lot/Serial Numbers | many to many | `stock.lot` | related through path `mo_id.lot_producing_ids` |
| `lot_id` | Lot/Serial Number | many to one | `stock.lot` | restricted by domain `[('product_id', '=', product_id),('id', 'in', lot_producing_ids)]`; must belong to the same company |
| `has_tracking` | Has Tracking | selection |  | read only; related through path `product_id.tracking` |
| `location_id` | Source Location | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; restricted by domain `[('usage','=','internal')]`; must belong to the same company; precomputed before insertion; Help: Location where the product you want to unbuild is. |
| `location_dest_id` | Destination Location | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; restricted by domain `[('usage','=','internal')]`; must belong to the same company; precomputed before insertion; Help: Location where you want to send the components resulting from the unbuild order. |
| `consume_line_ids` | Consumed Disassembly Lines | one to many | `stock.move` | read only; inverse field `consume_unbuild_id` |
| `produce_line_ids` | Processed Disassembly Lines | one to many | `stock.move` | read only; inverse field `unbuild_id` |
| `state` | Status | selection |  | default `draft` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `done` | Done |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_qty_positive` | Constraint | `check (product_qty > 0)` | The quantity to unbuild must be positive! | `mrp` |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_product_uom_id` | computation | self | `mrp` | depends: `mo_id`, `product_id` |  |
| `_compute_location_id` | computation | self | `mrp` | depends: `company_id` |  |
| `_compute_bom_id` | computation | self | `mrp` | depends: `mo_id`, `product_id`, `company_id` |  |
| `_compute_product_id` | computation | self | `mrp` | depends: `mo_id` |  |
| `_compute_product_qty` | computation | self | `mrp` | depends: `mo_id` |  |
| `create` | lifecycle override | self, vals_list | `mrp` | model_create_multi |  |
| `_unlink_except_done` | internal rule | self | `mrp` | ondelete |  |
| `_prepare_finished_move_line_vals` | preparation rule | self, finished_move | `mrp` |  |  |
| `_prepare_move_line_vals` | preparation rule | self, move, origin_move_line, taken_quantity | `mrp` |  |  |
| `action_unbuild` | user action | self | `mrp` |  |  |
| `_generate_consume_moves` | internal rule | self | `mrp` |  |  |
| `_generate_produce_moves` | internal rule | self | `mrp` |  |  |
| `_generate_move_from_existing_move` | internal rule | self, move, factor, location_id, location_dest_id | `mrp` |  |  |
| `_generate_move_from_bom_line` | internal rule | self, product, product_uom, quantity, bom_line_id, byproduct_id | `mrp` |  |  |
| `action_validate` | user action | self | `mrp` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_done` | UserError | You cannot delete an unbuild order if the state is 'Done'. | `mrp` |
| `action_unbuild` | UserError | You should provide a lot number for the final product. | `mrp` |
| `action_unbuild` | UserError | You cannot unbuild a undone manufacturing order. | `mrp` |
| `action_unbuild` | UserError | error_message | `mrp` |
| `action_unbuild` | UserError | error_message | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `group_mrp_manager` | yes | yes | yes | yes | `mrp` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_unbuild_search_view` | search |  | `product_id`, `mo_id` |  | `Draft`, `Done`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Product`, `Manufacturing Order` | `mrp` |
| `mrp.mrp_unbuild_kanban_view` | kanban |  | `name`, `product_qty`, `product_uom_id`, `product_id`, `state` |  |  | `mrp` |
| `mrp.mrp_unbuild_form_view` | form |  | `company_id`, `state`, `name`, `product_id`, `mo_bom_id`, `bom_id`, `product_qty`, `product_uom_id`, `mo_id`, `location_id`, `location_dest_id`, `has_tracking`, `lot_id`, `company_id` | `Unbuild`, `%(action_mrp_unbuild_moves)d` |  | `mrp` |
| `mrp.mrp_unbuild_form_view_simplified` | form |  | `company_id`, `state`, `product_id`, `bom_id`, `product_qty`, `product_uom_id`, `mo_id`, `location_id`, `location_dest_id`, `has_tracking`, `lot_id`, `company_id` | `Unbuild`, `Discard` |  | `mrp` |
| `mrp.mrp_unbuild_tree_view` | list |  | `name`, `product_id`, `bom_id`, `mo_id`, `lot_id`, `product_qty`, `product_uom_id`, `location_id`, `activity_exception_decoration`, `company_id`, `state` |  |  | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_unbuild` | Unbuild Orders | list,kanban,form,activity |  |  |  | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.unbuild.json`; views: `../../../schemas/interfaces/views/mrp.unbuild.json`.
