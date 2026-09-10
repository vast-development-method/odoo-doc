# Product Template Attribute Exclusion (`product.template.attribute.exclusion`)

**Transport name:** `product.template.attribute.exclusion`  
**Storage name:** `product_template_attribute_exclusion`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `point_of_sale`

Description: Product Template Attribute Exclusion

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `product_tmpl_id, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_template_attribute_value_id` | Attribute Value | many to one | `product.template.attribute.value` | indexed; on delete of the target: cascade |
| `product_tmpl_id` | Product Template | many to one | `product.template` | required; indexed; on delete of the target: cascade |
| `value_ids` | Attribute Values | many to many | `product.template.attribute.value` | restricted by domain `[('product_tmpl_id', '=', product_tmpl_id), ('ptav_active', '=', True)]`; association table `product_attr_exclusion_value_ids_rel` |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `product` | model_create_multi |  |
| `unlink` | lifecycle override | self | `product` |  |  |
| `write` | lifecycle override | self, vals | `product` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

Machine-readable definition: `../../../schemas/data/entities/product.template.attribute.exclusion.json`.
