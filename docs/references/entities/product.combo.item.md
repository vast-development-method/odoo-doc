# Product Combo Item (`product.combo.item`)

**Transport name:** `product.combo.item`  
**Storage name:** `product_combo_item`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `point_of_sale`

Description: Product Combo Item

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one |  | related through path `combo_id.company_id` and stored; precomputed before insertion |
| `combo_id` | Combo | many to one | `product.combo` | required; indexed; on delete of the target: cascade |
| `product_id` | Options | many to one | `product.product` | required; on delete of the target: restrict; restricted by domain `[["type", "!=", "combo"]]`; must belong to the same company |
| `currency_id` | Currency | many to one | `res.currency` | related through path `product_id.currency_id` |
| `lst_price` | Original Price | float |  | related through path `product_id.lst_price` |
| `extra_price` | Extra Price | float |  | default  |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_product_id_no_combo` | validation | self | `product` | constrains: `product_id` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_data_read` | internal rule | self, records, config | `point_of_sale` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_product_id_no_combo` | ValidationError | A combo choice can't contain products of type "combo". | `product` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |

Machine-readable definition: `../../../schemas/data/entities/product.combo.item.json`.
