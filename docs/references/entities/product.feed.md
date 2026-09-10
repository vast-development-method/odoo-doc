# Product Feed (`product.feed`)

**Transport name:** `product.feed`  
**Storage name:** `product_feed`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_sale`  
**Extended by packages:** `website_sale_stock`

Description: Product Feed

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `website_id` | Website | many to one | `website` | required |
| `pricelist_id` | Pricelist | many to one | `product.pricelist` | restricted by domain `[('website_id', 'in', (False, website_id)), ('selectable', '=', True)]`; Help: Specify a pricelist to localize the feed with a specific currency. If not set, the default website pricelist will be used. Note that the pricelist must be selectable on the website. |
| `lang_id` | Language | many to one | `res.lang` | required; computed by rule `_compute_lang_id` and stored; restricted by domain `[('id', 'in', website_lang_ids)]`; precomputed before insertion; Help: Select the language to translate product names, descriptions, and other text in the feed. |
| `website_lang_ids` | Website Lang | many to many |  | related through path `website_id.language_ids` |
| `product_category_ids` | Categories | many to many | `product.public.category` |  |
| `target` | Target | selection |  | required; default `gmc` |
| `access_token` | Access Token | single line text |  | required; read only; default computed dynamically (lambda _: uuid.uuid4().hex); not copied on duplication |
| `url` | Uniform resource locator | single line text |  | computed by rule `_compute_url` (not stored) |
| `last_notification_date` | Last Notification Date | date |  |  |
| `feed_cache` | Feed Cache | binary |  | read only; computed by rule `_compute_feed_cache` and stored |
| `cache_expiry` | Cache Expiry | date and time |  | required; read only; default computed dynamically (fields.Datetime.now) |

## Selection values

### `target` (Target)

| Value | Label |
|---|---|
| `gmc` | Google Merchant Center |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_url` | computation | self | `website_sale` | depends: `target` | Compute the full feed url. |
| `_compute_lang_id` | computation | self | `website_sale` | depends: `website_id` |  |
| `_compute_feed_cache` | computation | self | `website_sale` | depends: `website_id`, `pricelist_id`, `lang_id`, `product_category_ids` | Invalidate cache on feed parameter changes. |
| `_check_product_limit` | validation | self | `website_sale` | constrains: `product_category_ids`, `website_id` | Add a soft limit on the number of products a feed can contain.  A strong limit of 6000 is applied during the feed rendering phase. |
| `action_invalidate_cache` | user action | self | `website_sale` |  |  |
| `_render_and_cache_compressed_gmc_feed` | internal rule | self | `website_sale` |  | Render and cache the Google Merchant Center feed.  This method ensures that the feed is rendered only once per day and caches the result. If the feed parameters change, the cache is invalidated, and the feed is re-rendered.  :raises LockError: If the feed is already being rendered by another request. :return: The rendered feed compressed using gzip. :rtype: bytes |
| `_render_gmc_feed` | internal rule | self | `website_sale` |  | Render the Google Merchant Center feed.  See also https://support.google.com/merchants/answer/7052112 for the XML format.  :return: The rendered XML feed. :rtype: str |
| `_prepare_gmc_items` | preparation rule | self | `website_sale` |  | Prepare Google Merchant Center items' fields.  See Google's (https://support.google.com/merchants/answer/7052112) documentation for more information about each field.  :return: a dictionary for each product in this recordset. :rtype: list[dict] |
| `_get_feed_product_domain` | preparation rule | self | `website_sale` |  |  |
| `_get_feed_products` | preparation rule | self | `website_sale` |  |  |
| `_prepare_gmc_identifier` | preparation rule | self, product | `website_sale` |  | Prepare the product identifiers for Google Merchant Center.  :return: The barcode of the product as GTIN :rtype: dict |
| `_prepare_gmc_image_links` | preparation rule | self, product, base_url | `website_sale` |  | Prepare the product image links for Google Merchant Center.  :return: The main product image link, and the extra images. No videos. :rtype: dict |
| `_prepare_gmc_price_info` | preparation rule | self, product | `website_sale` |  | Prepare price-related information for Google Merchant Center.  Note: If the product is flagged to prevent zero price sales, an empty dictionary is returned.  :return: A dictionary containing nothing if the product is "prevent zero price sale", or:     - List price,     - Sale price (if applicable), and     - Comparison prices (e.g., $100 / ml) if "Product Reference Price" is enabled. :rtype: dict |
| `_prepare_gmc_stock_info` | preparation rule | self, _product | `website_sale_stock`, `website_sale` |  | Intended to be overridden in stock. |
| `_prepare_gmc_additional_info` | preparation rule | self, product | `website_sale` |  |  |
| `_notify_website_manager` | internal rule | self, **kwargs | `website_sale` |  | Send a notification to the website manager using OdooBot.  This method wraps around `message_notify` to notify the manager of the feed's website.  :param dict kwargs: Additional arguments passed to `message_notify`. :return: The created `mail.message` record. :rtype: mail.message |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_product_limit` | ValidationError | A single feed cannot contain more than %(limit)s products. Please separate products with Categories. | `website_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `website_sale` |
| `website.group_website_designer` | yes | yes | yes | yes | `website_sale` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_sale.product_feed_search` | search |  | `website_id`, `pricelist_id`, `lang_id`, `product_category_ids` |  | `Website`, `Pricelist`, `Language`, `eCommerce Category` | `website_sale` |
| `website_sale.product_feed_list` | list |  | `name`, `website_id`, `pricelist_id`, `lang_id`, `product_category_ids`, `url` |  |  | `website_sale` |
| `website_sale.product_feed_form` | form |  | `name`, `url`, `website_id`, `pricelist_id`, `lang_id`, `product_category_ids` |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_sale.action_product_feeds` | Product Feeds | list,form |  |  |  | `website_sale` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `website_sale.menu_product_feeds` |  |  | `website_sale.action_product_feeds` | 150 | `website_sale.group_product_feed` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `website_sale.action_invalidate_cache` | Reset Cache | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/product.feed.json`; views: `../../../schemas/interfaces/views/product.feed.json`.
