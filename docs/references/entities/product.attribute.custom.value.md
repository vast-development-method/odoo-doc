# Product Attribute Custom Value (`product.attribute.custom.value`)

**Transport name:** `product.attribute.custom.value`  
**Storage name:** `product_attribute_custom_value`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `sale`, `point_of_sale`

Description: Product Attribute Custom Value

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `custom_product_template_attribute_value_id, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_name` (not stored) |
| `custom_product_template_attribute_value_id` | Attribute Value | many to one | `product.template.attribute.value` | required; on delete of the target: restrict |
| `custom_value` | Custom Value | single line text |  |  |
| `sale_order_line_id` | Sales Order Line | many to one | `sale.order.line` | indexed (btree_not_null); on delete of the target: cascade |
| `pos_order_line_id` | PoS Order Line | many to one | `pos.order.line` | indexed (btree_not_null); on delete of the target: cascade |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_sol_custom_value_unique` | Constraint | `unique(custom_product_template_attribute_value_id, sale_order_line_id)` | Only one Custom Value is allowed per Attribute Value per Sales Order Line. | `sale` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_name` | computation | self | `product` | depends: `custom_product_template_attribute_value_id.name`, `custom_value` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `sales_team.group_sale_salesman` | yes | yes | yes | yes | `sale` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| product.attribute.custom.value: portal/public/employee read own records only | `[(4, ref('base.group_portal')), (4, ref('base.group_public')), (4, ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| product.attribute.custom.value: sales roles read all records | `[(4, ref('sales_team.group_sale_salesman')), (4, ref('product.group_product_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/product.attribute.custom.value.json`.
