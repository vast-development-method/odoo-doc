# Stock Landed Cost (`stock.landed.cost`)

**Transport name:** `stock.landed.cost`  
**Storage name:** `stock_landed_cost`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock_landed_costs`  
**Extended by packages:** `mrp_landed_costs`, `mrp_subcontracting_landed_costs`

Description: Stock Landed Cost

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `date desc, id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | read only; default computed dynamically (lambda self: _('New')); changes are tracked in the message thread; not copied on duplication |
| `date` | Date | date |  | required; default computed dynamically (fields.Date.context_today); changes are tracked in the message thread; not copied on duplication |
| `target_model` | Apply On | selection |  | required; default `picking`; not copied on duplication; on delete of the target: {"manufacturing": "set default"}; extended by packages `mrp_landed_costs` |
| `picking_ids` | Transfers | many to many | `stock.picking` | not copied on duplication |
| `cost_lines` | Cost Lines | one to many | `stock.landed.cost.lines` | inverse field `cost_id` |
| `valuation_adjustment_lines` | Valuation Adjustments | one to many | `stock.valuation.adjustment.lines` | inverse field `cost_id` |
| `description` | Item Description | multi line text |  |  |
| `amount_total` | Total | monetary |  | computed by rule `_compute_total_amount` and stored; changes are tracked in the message thread |
| `state` | State | selection |  | read only; default `draft`; changes are tracked in the message thread; not copied on duplication |
| `account_move_id` | Journal Entry | many to one | `account.move` | read only; indexed (btree_not_null); not copied on duplication |
| `account_journal_id` | Account Journal | many to one | `account.journal` | required; default computed dynamically (lambda self: self._default_account_journal_id()) |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `vendor_bill_id` | Vendor Bill | many to one | `account.move` | indexed (btree_not_null); not copied on duplication; restricted by domain `[["move_type", "=", "in_invoice"]]` |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `mrp_production_ids` | Manufacturing order | many to many | `mrp.production` | not copied on duplication; visible only to groups `stock.group_stock_manager` |

## Selection values

### `target_model` (Apply On)

| Value | Label |
|---|---|
| `picking` | Transfers |
| `manufacturing` | Manufacturing Orders |

### `state` (State)

| Value | Label |
|---|---|
| `draft` | Draft |
| `done` | Posted |
| `cancel` | Cancelled |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_account_journal_id` | preparation rule | self | `stock_landed_costs` |  | Take the journal configured in the company, else fallback on the stock journal. |
| `_compute_total_amount` | computation | self | `stock_landed_costs` | depends: `cost_lines.price_unit` |  |
| `_onchange_target_model` | on change | self | `mrp_landed_costs`, `stock_landed_costs` | onchange: `target_model` |  |
| `create` | lifecycle override | self, vals_list | `stock_landed_costs` | model_create_multi |  |
| `unlink` | lifecycle override | self | `stock_landed_costs` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `stock_landed_costs` |  |  |
| `button_cancel` | user action | self | `stock_landed_costs` |  |  |
| `button_validate` | user action | self | `stock_landed_costs` |  |  |
| `get_valuation_lines` | operation | self | `stock_landed_costs` |  |  |
| `compute_landed_cost` | operation | self | `stock_landed_costs` |  |  |
| `_get_targeted_move_ids` | preparation rule | self | `mrp_landed_costs`, `mrp_subcontracting_landed_costs`, `stock_landed_costs` |  |  |
| `_check_can_validate` | validation | self | `stock_landed_costs` |  |  |
| `_check_sum` | validation | self | `stock_landed_costs` |  | Check if each cost line its valuation lines sum to the correct amount and if the overall total amount is correct also |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `button_cancel` | UserError | Validated landed costs cannot be cancelled, but you could create negative landed costs to reverse them | `stock_landed_costs` |
| `button_validate` | UserError | Cost and adjustments lines do not match. You should maybe recompute the landed costs. | `stock_landed_costs` |
| `get_valuation_lines` | UserError | You cannot apply landed costs on the chosen %s(s). Landed costs can only be applied for products with FIFO or average costing method. | `stock_landed_costs` |
| `_check_can_validate` | UserError | Only draft landed costs can be validated | `stock_landed_costs` |
| `_check_can_validate` | UserError | Please define %s on which those additional costs should apply. | `stock_landed_costs` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock_landed_costs` |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp_landed_costs.view_mrp_landed_costs_form` | field | `stock_landed_costs.view_stock_landed_cost_form` | `target_model` |  |  | `mrp_landed_costs` |
| `mrp_subcontracting_landed_costs.view_mrp_landed_costs_form` | field | `mrp_landed_costs.view_mrp_landed_costs_form` | `mrp_production_ids` |  |  | `mrp_subcontracting_landed_costs` |
| `stock_landed_costs.view_stock_landed_cost_form` | form |  | `company_id`, `state`, `name`, `date`, `target_model`, `picking_ids`, `account_journal_id`, `company_id`, `account_move_id`, `vendor_bill_id`, `cost_lines`, `product_id`, `price_unit`, `currency_id`, `split_method`, `account_id`, `name`, `product_id`, `name`, `account_id`, `split_method`, `price_unit`, `currency_id`, `currency_id`, `amount_total`, `valuation_adjustment_lines`, `product_id`, `quantity`, `currency_id`, `former_cost`, `additional_landed_cost`, `cost_line_id`, `product_id`, `weight`, `volume`, `quantity`, `currency_id`, `former_cost`, `final_cost`, `additional_landed_cost` | `Validate`, `Cancel`, `Compute` |  | `stock_landed_costs` |
| `stock_landed_costs.view_stock_landed_cost_tree` | list |  | `name`, `date`, `company_id`, `state`, `activity_exception_decoration` |  |  | `stock_landed_costs` |
| `stock_landed_costs.view_stock_landed_cost_tree2` | list |  | `name`, `date`, `currency_id`, `amount_total`, `state`, `company_id` |  |  | `stock_landed_costs` |
| `stock_landed_costs.stock_landed_cost_view_kanban` | kanban |  | `name`, `state`, `date`, `account_journal_id` |  |  | `stock_landed_costs` |
| `stock_landed_costs.view_stock_landed_cost_search` | search |  | `name`, `picking_ids` |  | `Draft`, `Done`, `Date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Status`, `Date` | `stock_landed_costs` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock_landed_costs.action_stock_landed_cost` | Landed Costs | list,form,kanban |  | `{}` |  | `stock_landed_costs` |

Machine-readable definition: `../../../schemas/data/entities/stock.landed.cost.json`; views: `../../../schemas/interfaces/views/stock.landed.cost.json`.
