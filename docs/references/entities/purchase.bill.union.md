# Purchases & Bills Union (`purchase.bill.union`)

**Transport name:** `purchase.bill.union`  
**Storage name:** `purchase_bill_union`  
**Kind:** persistent entity (one table)  
**Defined by package:** `purchase`

Description: Purchases & Bills Union

## Identity and behavior

- Default ordering: `date desc, name desc`
- Display name search fields: `["name", "reference"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reference | single line text |  | read only |
| `reference` | Source | single line text |  | read only |
| `partner_id` | Vendor | many to one | `res.partner` | read only |
| `date` | Date | date |  | read only |
| `amount` | Amount | float |  | read only |
| `currency_id` | Currency | many to one | `res.currency` | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `vendor_bill_id` | Vendor Bill | many to one | `account.move` | read only |
| `purchase_order_id` | Purchase Order | many to one | `purchase.order` | read only |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `purchase` |  |  |
| `_compute_display_name` | computation | self | `purchase` | depends: `currency_id`, `reference`, `amount`, `purchase_order_id`; depends_context: `show_total_amount` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `purchase.group_purchase_user` | no | yes | no | no | `purchase` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Purchases & Bills Union multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase.view_purchase_bill_union_filter` | search |  | `name`, `amount`, `partner_id` |  | `Purchase Orders`, `Vendor Bills` | `purchase` |
| `purchase.view_purchase_bill_union_tree` | list |  | `name`, `reference`, `partner_id`, `date`, `amount`, `currency_id`, `company_id` |  |  | `purchase` |

Machine-readable definition: `../../../schemas/data/entities/purchase.bill.union.json`; views: `../../../schemas/interfaces/views/purchase.bill.union.json`.
