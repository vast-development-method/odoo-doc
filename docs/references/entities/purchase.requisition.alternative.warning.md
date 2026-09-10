# Wizard in case purchase order still has open alternative requests for quotation (`purchase.requisition.alternative.warning`)

**Transport name:** `purchase.requisition.alternative.warning`  
**Storage name:** `purchase_requisition_alternative_warning`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `purchase_requisition`

Description: Wizard in case PO still has open alternative requests for quotation

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `po_ids` | purchase orders to Confirm | many to many | `purchase.order` | association table `warning_purchase_order_rel` |
| `alternative_po_ids` | Alternative purchase orders | many to many | `purchase.order` | association table `warning_purchase_order_alternative_rel` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_keep_alternatives` | user action | self | `purchase_requisition` |  |  |
| `action_cancel_alternatives` | user action | self | `purchase_requisition` |  |  |
| `_action_done` | internal rule | self | `purchase_requisition` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase_requisition.purchase_requisition_alternative_warning_form` | form |  | `alternative_po_ids`, `currency_id`, `partner_id`, `name`, `date_planned`, `amount_total`, `state` | `Cancel Alternatives`, `Keep Alternatives`, `Discard` |  | `purchase_requisition` |

Machine-readable definition: `../../../schemas/data/entities/purchase.requisition.alternative.warning.json`; views: `../../../schemas/interfaces/views/purchase.requisition.alternative.warning.json`.
