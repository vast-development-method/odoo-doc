# Purchase Requisition (`purchase.requisition`)

**Transport name:** `purchase.requisition`  
**Storage name:** `purchase_requisition`  
**Kind:** persistent entity (one table)  
**Defined by package:** `purchase_requisition`  
**Extended by packages:** `purchase_requisition_stock`

Description: Purchase Requisition

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Agreement | single line text |  | required; read only; default computed dynamically (lambda self: _('New')); not copied on duplication |
| `active` | Active | boolean |  | default `True` |
| `reference` | Reference | single line text |  |  |
| `order_count` | Number of Orders | integer |  | computed by rule `_compute_orders_number` (not stored) |
| `vendor_id` | Vendor | many to one | `res.partner` | must belong to the same company |
| `requisition_type` | Agreement Type | selection |  | required; default `blanket_order` |
| `date_start` | Start Date | date |  | changes are tracked in the message thread |
| `date_end` | End Date | date |  | changes are tracked in the message thread |
| `user_id` | Purchase Representative | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); must belong to the same company |
| `description` | Description | rich text |  |  |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `purchase_ids` | Purchase Orders | one to many | `purchase.order` | inverse field `requisition_id` |
| `line_ids` | Products to Purchase | one to many | `purchase.requisition.line` | inverse field `requisition_id` |
| `product_id` | Product | many to one | `product.product` | related through path `line_ids.product_id` |
| `state` | Status | selection |  | required; default `draft`; changes are tracked in the message thread; not copied on duplication |
| `currency_id` | Currency | many to one | `res.currency` | required; computed by rule `_compute_currency_id` and stored; precomputed before insertion |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | restricted by domain `[('company_id', '=', company_id)]` |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | required; default computed dynamically (_default_picking_type_id); restricted by domain `['\|',('warehouse_id', '=', False), ('warehouse_id.company_id', '=', company_id)]` |

## Selection values

### `requisition_type` (Agreement Type)

| Value | Label |
|---|---|
| `blanket_order` | Blanket Order |
| `purchase_template` | Purchase Template |

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `confirmed` | Confirmed |
| `done` | Closed |
| `cancel` | Cancelled |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_vendor` | on change | self | `purchase_requisition` | onchange: `vendor_id` |  |
| `_compute_currency_id` | computation | self | `purchase_requisition` | depends: `vendor_id` |  |
| `_compute_orders_number` | computation | self | `purchase_requisition` | depends: `purchase_ids` |  |
| `_check_dates` | validation | self | `purchase_requisition` | constrains: `date_start`, `date_end` |  |
| `create` | lifecycle override | self, vals_list | `purchase_requisition` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `purchase_requisition` |  |  |
| `unlink` | lifecycle override | self | `purchase_requisition` |  |  |
| `action_cancel` | user action | self | `purchase_requisition` |  |  |
| `action_confirm` | user action | self | `purchase_requisition` |  |  |
| `action_draft` | user action | self | `purchase_requisition` |  |  |
| `action_done` | user action | self | `purchase_requisition` |  | Generate all purchase order based on selected lines, should only be called on one agreement at a time |
| `_unlink_if_draft_or_cancel` | internal rule | self | `purchase_requisition` | ondelete |  |
| `_default_picking_type_id` | preparation rule | self | `purchase_requisition_stock` |  |  |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_dates` | ValidationError | End date cannot be earlier than start date. Please check dates for agreements: %s | `purchase_requisition` |
| `write` | UserError | You cannot change the Agreement Type or Company of a not draft purchase agreement. | `purchase_requisition` |
| `action_confirm` | UserError | You cannot confirm agreement '%(agreement)s' because it does not contain any product lines. | `purchase_requisition` |
| `action_confirm` | UserError | You cannot confirm a blanket order with lines missing a price. | `purchase_requisition` |
| `action_confirm` | UserError | You cannot confirm a blanket order with lines missing a quantity. | `purchase_requisition` |
| `action_done` | UserError | To close this purchase requisition, cancel related Requests for Quotation.  Imagine the mess if someone confirms these duplicates: double the order, double the trouble :) | `purchase_requisition` |
| `_unlink_if_draft_or_cancel` | UserError | You can only delete draft or cancelled requisitions. | `purchase_requisition` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |
| `purchase.group_purchase_manager` | no | yes | no | no | `purchase_requisition` |
| `stock.group_stock_manager` | yes | yes | no | no | `purchase_requisition_stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Purchase Requisition multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase_requisition.view_purchase_requisition_form` | form |  | `company_id`, `currency_id`, `active`, `state`, `order_count`, `name`, `vendor_id`, `user_id`, `requisition_type`, `currency_id`, `date_start`, `date_end`, `reference`, `company_id`, `line_ids`, `product_id`, `product_description_variants`, `product_qty`, `qty_ordered`, `product_uom_id`, `analytic_distribution`, `price_unit`, `product_id`, `product_qty`, `qty_ordered`, `product_uom_id`, `analytic_distribution`, `company_id`, `description` | `New Quotation`, `Confirm`, `Close`, `Reset to Draft`, `Cancel`, `%(action_purchase_requisition_list)d` |  | `purchase_requisition` |
| `purchase_requisition.view_purchase_requisition_tree` | list |  | `message_needaction`, `name`, `vendor_id`, `requisition_type`, `user_id`, `company_id`, `date_start`, `date_end`, `reference`, `state`, `activity_exception_decoration` |  |  | `purchase_requisition` |
| `purchase_requisition.view_purchase_requisition_kanban` | kanban |  | `name`, `state`, `requisition_type`, `vendor_id`, `user_id` |  |  | `purchase_requisition` |
| `purchase_requisition.view_purchase_requisition_filter` | search |  | `vendor_id`, `name`, `user_id`, `product_id` |  | `My Agreements`, `Blanket Orders`, `Purchase Templates`, `Draft`, `Done`, `Archived`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Purchase Representative`, `Status`, `Ordering Date` | `purchase_requisition` |
| `purchase_requisition_stock.view_purchase_requisition_form_inherit` | field | `purchase_requisition.view_purchase_requisition_form` | `reference`, `picking_type_id` |  |  | `purchase_requisition_stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `purchase_requisition.action_purchase_requisition` | Purchase Agreements | list,kanban,form |  | `{}` |  | `purchase_requisition` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `purchase_requisition.action_report_purchase_requisitions` | Purchase Agreements | qweb-pdf | `purchase_requisition.report_purchaserequisitions` | `'Purchase Agreement - %s' % (object.name)` |  |

Machine-readable definition: `../../../schemas/data/entities/purchase.requisition.json`; views: `../../../schemas/interfaces/views/purchase.requisition.json`.
