# Removal Strategy (`product.removal`)

**Transport name:** `product.removal`  
**Storage name:** `product_removal`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `point_of_sale`

Description: Removal Strategy

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `method` | Method | single line text |  | required; translatable; Help: FIFO, LIFO... |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.view_removal` | form |  | `name`, `method` |  |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/product.removal.json`; views: `../../../schemas/interfaces/views/product.removal.json`.
