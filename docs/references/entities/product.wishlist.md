# Product Wishlist (`product.wishlist`)

**Transport name:** `product.wishlist`  
**Storage name:** `product_wishlist`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_sale_wishlist`  
**Extended by packages:** `website_sale_stock_wishlist`

Description: Product Wishlist

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Owner | many to one | `res.partner` | indexed (btree_not_null) |
| `product_id` | Product | many to one | `product.product` | required |
| `currency_id` | Currency | many to one | `res.currency` | read only; related through path `website_id.currency_id` |
| `pricelist_id` | Pricelist | many to one | `product.pricelist` | Help: Pricelist when added |
| `price` | Price | monetary |  | currency taken from `currency_id`; Help: Price of the product when it has been added in the wishlist |
| `website_id` | Website | many to one | `website` | required; on delete of the target: cascade |
| `active` | Active | boolean |  | required; default `True` |
| `stock_notification` | Stock Notification | boolean |  | required; computed by rule `_compute_stock_notification` (not stored); default  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_product_unique_partner_id` | Constraint | `UNIQUE(product_id, partner_id)` | Duplicated wishlisted product for this partner. | `website_sale_wishlist` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `current` | operation | self | `website_sale_wishlist` | model | Get all wishlist items that belong to current user or session, filter products that are unpublished. |
| `_add_to_wishlist` | internal rule | self, pricelist_id, currency_id, website_id, price, product_id, partner_id | `website_sale_wishlist` | model |  |
| `_check_wishlist_from_session` | validation | self | `website_sale_wishlist` | model | Assign all wishlist withtout partner from this the current session |
| `_gc_sessions` | background operation | self, *args, **kwargs | `website_sale_wishlist` | autovacuum | Remove wishlists for unexisting sessions. |
| `_compute_stock_notification` | computation | self | `website_sale_stock_wishlist` | depends: `product_id`, `partner_id` |  |
| `_inverse_stock_notification` | inverse computation | self | `website_sale_stock_wishlist` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `website_sale_wishlist` |
| `base.group_public` | no | no | no | no | `website_sale_wishlist` |
| `base.group_portal` | yes | yes | yes | yes | `website_sale_wishlist` |
| `base.group_user` | yes | yes | yes | yes | `website_sale_wishlist` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| See own Wishlist | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('partner_id','=', user.partner_id.id)]` | True | True | True | True |
| See all wishlist | `[(4, ref('sales_team.group_sale_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/product.wishlist.json`.
