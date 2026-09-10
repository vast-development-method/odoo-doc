# Update product attribute value (`update.product.attribute.value`)

**Transport name:** `update.product.attribute.value`  
**Storage name:** `update_product_attribute_value`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `product`

Description: Update product attribute value

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `attribute_value_id` | Attribute Value | many to one | `product.attribute.value` | required |
| `mode` | Mode | selection |  |  |
| `message` | Message | single line text |  | computed by rule `_compute_message` (not stored) |
| `product_count` | Product Count | integer |  | computed by rule `_compute_product_count` (not stored) |

## Selection values

### `mode` (Mode)

| Value | Label |
|---|---|
| `add` | Add to existing products |
| `update_extra_price` | Update the extra price on existing products |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_message` | computation | self | `product` | depends: `product_count`, `mode`, `attribute_value_id` |  |
| `_compute_product_count` | computation | self | `product` | depends: `mode` |  |
| `action_confirm` | user action | self | `product` |  |  |
| `_add_value_to_existing_attribute_lines` | internal rule | self | `product` |  |  |
| `_update_extra_price_on_existing_products` | internal rule | self | `product` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_product_manager` | yes | yes | yes | no | `product` |
| `stock.group_stock_manager` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product.update_product_attribute_value_form` | form |  | `product_count`, `mode`, `message` | `Cancel`, `Confirm` |  | `product` |

Machine-readable definition: `../../../schemas/data/entities/update.product.attribute.value.json`; views: `../../../schemas/interfaces/views/update.product.attribute.value.json`.
