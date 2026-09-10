# Lunch Extras (`lunch.topping`)

**Transport name:** `lunch.topping`  
**Storage name:** `lunch_topping`  
**Kind:** persistent entity (one table)  
**Defined by package:** `lunch`

Description: Lunch Extras

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `price` | Price | monetary |  | required |
| `supplier_id` | Supplier | many to one | `lunch.supplier` | indexed (btree_not_null); on delete of the target: cascade |
| `topping_category` | Topping Category | integer |  | required; default `1` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `lunch` | depends: `price`; depends_context: `company` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_lunch_user` | no | yes | no | no | `lunch` |
| `group_lunch_manager` | yes | yes | yes | yes | `lunch` |

Machine-readable definition: `../../../schemas/data/entities/lunch.topping.json`.
