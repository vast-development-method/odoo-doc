# Interfaces

The request endpoints, named operations, screens, generated documents, notifications, external
integrations and import and export contracts of this folder. Screens are described as procedures on
views, with the fields shown, the buttons and their guards, the filters and the groupings, without
reference to any client technology.

---

## 1. Conventions

| Term | Meaning |
|---|---|
| public | The endpoint may be called without authentication; the current user is then the site's public user. |
| signed in | The endpoint requires an authenticated user. |
| page request | The endpoint answers with a rendered page, a redirect or a status code. |
| data request | The endpoint answers with a structured payload. |
| site-scoped | The endpoint resolves the current site from the request host and the language prefix. |
| read only | The endpoint promises not to write; a rebuild may route it to a read replica. |
| retrieval request | The endpoint is reached with the retrieval method of the web request protocol: the request carries no body, its parameters travel in the query string, and repeating it has no further effect. |
| submission request | The endpoint is reached with the submission method of the web request protocol: the request carries a body, and repeating it repeats its effect. |
| external wire literal | A path, a query parameter or a payload field name that an outside party already writes a certain way, reproduced here exactly and glossed in full words. |

Address patterns are written with the placeholder inside angle brackets. A placeholder that names a record
is a slug: the readable label, a hyphen and the numeric identifier
([calculations.md](calculations.md) §3.1).

Every page request under the shop path first evaluates the shop-access guard WS-290; every page request
evaluates the site resolution of [multi-site-and-languages.md](multi-site-and-languages.md) §1 and the
language handling of §5 of the same file.

---

## 2. Site endpoints

### 2.1 Core

| Path | Kind | Access | Purpose |
|---|---|---|---|
| `/` | page | public, site-scoped | The home page (§10 of [workflows.md](workflows.md)). |
| `/@/` and `/@/<path>` | page | signed in | Opens a public path inside the editing shell. |
| `/pages` | page | Editor and Designer | The page manager listing, which merges Website Pages and Website Technical Pages. |
| `/robots.txt` | page, plain text | public, site-scoped | The crawler exclusion response. |
| `/sitemap.xml` | page | public, site-scoped | The site index. |
| `/favicon.ico` | page | public, site-scoped | The site icon. |
| `/google<sixteen characters>.html` | page | public, site-scoped | The search console verification response (WS-156). |
| `/website/lang/<prefix>` | page | public, site-scoped | Switches the display language and sets the cookie. |
| `/website/force/<site identifier>` | page | multi-site Restricted Editor | Forces another site into the session, hopping to its domain first when needed. |
| `/website/info` | page | public | The public information page of the installation. |
| `/website/iframefallback` | page | public | The fallback document used while a page is loaded inside the editor. |
| `/website/social/<network>` | page | public, site-scoped | Redirects to the site's account on that network. |
| `/website/country_infos/<country>` | data | public | The address field layout, the state list and the postal-code requirement of a country. |
| `/website/get_languages` | data | public | The site's languages with their prefixes. |
| `/website/get_current_currency` | data | public | The display currency identifier, symbol and position. |
| `/website/save_session_layout_mode` | data | public | Remembers a grid-or-list choice in the session. |
| `/website/google_maps_api_key` | data | public | The mapping service access key of the site. |

### 2.2 Editing and design

All of the following require the Restricted Editor group at least; those that create or delete structural
records require the Editor and Designer group.

| Path | Access | Effect |
|---|---|---|
| `/website/add` | Editor and Designer | Creates a page from a template, optionally with a menu entry (§5 of [workflows.md](workflows.md)). |
| `/website/get_new_page_templates` | Editor and Designer | The template picker content, grouped, including pages flagged as offered templates. |
| `/website/check_new_content_access_rights` | Restricted Editor | Which kinds of new content the caller may create. |
| `/website/save_xml` | Restricted Editor | Saves one edited region into a template architecture (§19 of [workflows.md](workflows.md)). |
| `/website/reset_template` | Editor and Designer | Soft or hard reset of a template architecture (§20 of [workflows.md](workflows.md)). |
| `/website/theme_customize_data`, `/website/theme_customize_data_get`, `/website/theme_customize_bundle_reload` | Restricted Editor | Enables and disables templates and assets, reads the current state, and rebuilds the compiled bundle. |
| `/website/theme_upload_font` | Editor and Designer | Uploads and validates a font (WS-196). |
| `/website/google_font_metadata` | Restricted Editor | The catalogue of the external font service. |
| `/website/get_seo_data`, `/website/seo_suggest` | write access on the record | Reads the search engine metadata of a record and suggests keywords. |
| `/website/get_alt_images`, `/website/update_alt_images` | Restricted Editor | Lists and rewrites the alternate text of the pictures of a page. |
| `/website/get_suggested_links`, `/website/check_existing_link` | Restricted Editor | Suggests link targets from the address enumeration and reports whether an address exists. |
| `/website/update_broken_links` | Editor and Designer | Rewrites the links of a page that point at a moved address. |
| `/website/update_footer_template` | Editor and Designer | Replaces the footer template of the site. |
| `/website/get_translated_elements` | Restricted Editor | The translatable elements of the current page for the translation editor. |
| `/website/check_can_modify_any` | Restricted Editor | Reports whether the caller may modify at least one record of a given set. |
| `/website/snippet/filters`, `/website/snippet/filter_templates`, `/website/snippet/options_filters` | Restricted Editor | The records a dynamic content block renders, the templates it may use and the filter list offered in the editor (WS-250 to WS-264). |
| `/website/snippet/autocomplete` | public | The autocompletion answer of the site search (§17 of [workflows.md](workflows.md)). |
| `/website/configurator` and its steps | Editor and Designer | The first-run configurator (§25 of [workflows.md](workflows.md)). |
| `/website/track_installing_modules` | Editor and Designer | Reports the progress of the packages the configurator installs. |
| `/website/fetch_dashboard_data` | Restricted Editor | The figures of the site dashboard. |
| `/web_unsplash/attachment/add` | Restricted Editor | Stores a picture chosen in the external picture library as an attachment of the site. |

### 2.3 Public forms

| Path | Kind | Access | Purpose |
|---|---|---|---|
| `/website/form` | page, submission request | public, site-scoped | The generic entry point of the form builder; the target entity travels in the body. |
| `/website/form/<entity name>` | page, submission request | public, site-scoped | The same with the target entity in the path. The last segment is the transport name of the target entity, reproduced byte for byte because it is what the page posts to. It answers the created identifier, the list of failed fields, or an error message (§27 of [workflows.md](workflows.md)). |
| `/website/action/<path segment or identifier>` | page | public when the action is published | Runs a published server action from the site (WS-623). |

### 2.4 Blog

| Path | Kind | Access | Purpose |
|---|---|---|---|
| `/blog`, `/blog/page/<page>` | page | public, site-scoped | The index of every blog of the site; with exactly one blog it redirects temporarily to that blog. |
| `/blog/<blog>`, `/blog/<blog>/page/<page>` | page | public, site-scoped | One blog's listing. |
| `/blog/tag/<tag>`, `/blog/tag/<tag>/page/<page>` | page | public, site-scoped | The listing filtered by a tag across blogs. |
| `/blog/<blog>/tag/<tag>`, `/blog/<blog>/tag/<tag>/page/<page>` | page | public, site-scoped | The same inside one blog. |
| `/blog/<blog>/<post>` | page | public, site-scoped | One post. |
| `/blog/<blog>/post/<post>` | page | public, site-scoped | The legacy form of the post address; answers a permanent redirect to the current form. |
| `/blog/<blog>/feed` | page, syndication document | public, site-scoped | The subscription feed (WS-542). |

### 2.5 Forum

| Path | Kind | Access | Purpose |
|---|---|---|---|
| `/forum` | page | public, site-scoped | The list of forums of the site. |
| `/forum/<forum>` | page | public, site-scoped | The question listing, with the sort, the tag filter and the pager. |
| `/forum/<forum>/ask` | page | reputation gate WS-581 | The ask form. |
| `/forum/<forum>/new` | page, submission request | reputation gate WS-581 | Creates a question. |
| `/forum/<forum>/<post>` | page | public, site-scoped | One question with its answers and comments. |
| `/forum/<forum>/post/<post>/edit` and `/save` | page | reputation gate WS-583 | Edits a post. |
| `/forum/<forum>/post/<post>/upvote`, `/downvote` | data | reputation gates WS-594 to WS-596 | Casts or withdraws a vote. |
| `/forum/<forum>/post/<post>/toggle_correct` | data | reputation gate WS-593 | Accepts or withdraws the acceptance of an answer. |
| `/forum/<forum>/post/<post>/comment` | page, submission request | reputation gate WS-587 | Posts a comment. |
| `/forum/<forum>/post/<post>/comment/<comment>/convert_to_answer`, `/delete` | page, data | reputation gates WS-588, WS-589 | Converts or deletes a comment. |
| `/forum/<forum>/post/<post>/convert_to_comment` | page | reputation gate WS-588 | Converts an answer into a comment. |
| `/forum/<forum>/post/<post>/delete` | data | reputation gate WS-592 | Archives a post. |
| `/forum/<forum>/post/<post>/flag`, `/mark_as_offensive`, `/ask_for_mark_as_offensive` | data, page | reputation gates WS-590, WS-591 | Flags a post and marks it offensive with a reason. |
| `/forum/<forum>/post/<post>/validate`, `/refuse` | data | reputation gate WS-591 | Moderates a pending or flagged post. |
| `/forum/<forum>/question/<question>/close`, `/ask_for_close`, `/reopen` | page, data | reputation gate WS-584 | Closes and reopens a question with a reason. |
| `/forum/<forum>/flagged_queue`, `/closed_posts`, `/offensive_posts` | page | reputation gate WS-591 | The three moderation queues. |
| `/forum/<forum>/faq/karma` | page | public | The reputation table of the forum, listing every threshold. |
| `/forum/<forum>/tag/<tag>/questions` | page | public | The questions carrying one tag. |
| `/forum/<forum>/partner/<contact identifier>` | page | public | Redirects to the public profile of a participant. |

### 2.6 Public directories and profiles

| Path | Kind | Access | Purpose |
|---|---|---|---|
| `/partners/<contact>` | page | public, site-scoped | The public page of one contact (WS-550, WS-551). |
| `/customers` and its faceted forms `/customers/country/<country>`, `/customers/tag/<tag>`, `/customers/page/<page>` | page | public, site-scoped | The customer reference directory, faceted by country, industry and public tag. |
| `/customers/<contact identifier>` | page | public, site-scoped | One reference, reached from the map. |
| `/google_map` | page | public, site-scoped | The map frame of at most the requested number of published contacts (WS-555). |
| `/profile/users` | page | public, site-scoped | The ranking of public profiles. |
| `/profile/user/<account identifier>` | page | public, gated by WS-557 | One public profile. |
| `/profile/user/save` | data | the owner, or an administrator | Saves the editable profile fields (WS-559). |
| `/profile/ranks_badges` | page | public | The rank and badge catalogue. |
| `/profile/send_validation_email`, `/profile/validate_email`, `/profile/validate_email/close` | data, page | the owner | The address validation flow (WS-560). |
| `/group/is_member` | data | public | Whether the current visitor belongs to a mailing group, used by the public group pages. |

### 2.7 Tracked links

| Path | Kind | Access | Purpose |
|---|---|---|---|
| `/r/<code>` | page | public, site-scoped | Records one click and answers a permanent redirect to the target (WS-621). |
| `/r/<code>+` | page | public | The statistics page of that link. |
| `/r` | page | signed in | The tracked-link workbench, where a link is created and its code and statistics are read. |
| `/website_links/new`, `/website_links/add_code`, `/website_links/recent_links` | data | signed in | Creates a tracked link, adds an alternative code and lists the recent links of the caller. |

---

## 3. Storefront endpoints

### 3.1 Catalogue

| Path | Kind | Access | Purpose | Request | Response |
|---|---|---|---|---|---|
| `/shop`, `/shop/page/<page>`, `/shop/category/<category>`, `/shop/category/<category>/page/<page>` | page | public, site-scoped | The shop listing | Query: the search term, the minimum and maximum price, the tags, the order, and the repeated attribute filter parameter | The listing page. An access failure answers "not found". A category given as a query parameter answers a permanent redirect. |
| `/shop/<product>`, `/shop/<category>/<product>` | page | public, site-scoped | The product page | Query: the price list and the attribute values | The product page, or a permanent redirect to the canonical address. A price list that is not a number raises `Wrong format: got \`pricelist=%s\`, expected an integer` |
| `/shop/product/<product>` | page | public, site-scoped | Compatibility address | Query: the category | A permanent redirect to the canonical address. |
| `/shop/<product template>/document/<document>` | page, read only | public, site-scoped | Downloads a published product document | none | The file as an attachment, or a redirect to the shop when the guard fails. |
| `/website_sale/get_combination_info` | data | public, site-scoped, read only | The combination information payload | the template, the variant, the combination, the quantity and the unit | The payload of [storefront-catalogue.md](storefront-catalogue.md) §8.4, with the internal keys stripped and the currency precision added. |
| `/sale/create_product_variant` | data | public | Creates or fetches the variant of a combination | the template and the attribute values | The variant identifier, or 0 for any failure. |
| `/shop/product/is_add_to_cart_allowed` | data | public, site-scoped, read only | Whether a variant may be added | the variant | A boolean, evaluated with elevated privileges. |
| `/shop/change_pricelist/<price list>` | page | public, site-scoped | Selects a price list | none | A redirect back to the referring page, with the price bounds converted when applicable. |
| `/shop/pricelist` | page | public, site-scoped | Applies or clears a promotional code | the code and the return path | A redirect to the return path, or to the return path carrying the code-not-available marker. |
| `/shop/save_shop_layout_mode` | data | public, site-scoped | Remembers the grid or list choice | the mode, `grid` or `list` | Nothing. Any other value is rejected. |
| `/shop/products/recently_viewed_update`, `/shop/products/recently_viewed_delete` | data | public, site-scoped | Records and forgets a product view | the variant, or the template for the deletion | An empty payload. |
| `/gmc.<extension>` (external wire literal) | page | public, site-scoped | The product syndication document | the feed identifier and the access token, both external wire literals | The compressed document; see §4.4. |

### 3.2 Product configurator

| Path | Kind | Access | Purpose |
|---|---|---|---|
| `/website_sale/should_show_product_configurator` | data, read only | public, site-scoped | Whether the configurator dialogue must open for a template, a combination and an "already configured" flag. |
| `/website_sale/product_configurator/get_values` | data, read only | public, site-scoped | The configurator payload for a template, with the display currency and price list injected. |
| `/website_sale/product_configurator/create_product` | data | public, site-scoped | Creates the variant of a chosen combination. |
| `/website_sale/product_configurator/update_combination` | data, read only | public, site-scoped | Re-prices a combination inside the dialogue. |
| `/website_sale/product_configurator/get_optional_products` | data, read only | public, site-scoped | The optional products offered for a combination. |
| `/website_sale/combo_configurator/get_data` | data, read only | public, site-scoped | The combo choices of a combo product, with internal references hidden. |
| `/website_sale/combo_configurator/get_price` | data, read only | public, site-scoped | The price of a combo selection. |

All of them inject the display currency and the request price list into the call, apply taxes to every
price and extra price they return, mark a product as not sellable when its price is zero while the site
forbids it, add a strikethrough price when one applies, and offer only optional products that may actually
be added to the cart on this site.

### 3.3 Cart

| Path | Kind | Access | Purpose | Response |
|---|---|---|---|---|
| `/shop/cart` | page | public, site-scoped | The cart page, including abandoned-cart revival | The cart page. |
| `/shop/cart/add` | data | public, site-scoped | Adds one product and its linked products | The cart quantity, the resulting quantity, the notification payload and the analytics payload; see [storefront-checkout.md](storefront-checkout.md) §2.1. |
| `/shop/cart/quick_add` | data | signed in, site-scoped | Adds and re-renders the cart blocks | The add payload plus the rendered cart lines, the short cart summary, the reorder history and the readiness flag. |
| `/shop/cart/update` | data | public, site-scoped | Changes one line quantity | The added quantity, the line, the resulting quantity, the warning, the cart quantity, the readiness flag and the amounts, plus the rendered cart lines, totals and reorder history. |
| `/shop/cart/quantity` | data | public, site-scoped | The header counter | A whole number. |
| `/shop/cart/clear` | data | public, site-scoped | Empties the cart | Nothing. |
| `/my/orders/reorder` | data | public, site-scoped | Adds every reorderable line of a past order | The analytics payload and the cart quantity; raises `Nothing can be reordered in this order` when nothing qualifies. |

### 3.4 Checkout and payment

| Path | Kind | Access | Purpose | Notes |
|---|---|---|---|---|
| `/shop/checkout` | page, retrieval request | public, site-scoped | The address and delivery step | The skip flag skips the page when nothing is required. |
| `/shop/address` | page, retrieval request | public, site-scoped | The address form | The contact, the address kind, the "use as both" flag and the callback. |
| `/shop/address/submit` | page, submission request | public, site-scoped | Creates or updates an address | Answers the redirect target on success, or the invalid fields and messages on failure. |
| `/shop/update_address` | data | public, site-scoped | Selects an existing address | The contact and the address kind. Answers "forbidden" for a contact the shopper may not use. |
| `/shop/extra_info` | page | public, site-scoped | The extra-information step | Redirects to the payment page when the page option is inactive. |
| `/website/form/shop.sale.order` (external wire literal) | page, submission request | public, site-scoped | Submits the extra-information form | The generic public form endpoint. Its last segment is the transport name of the target entity, which for this form is the Sales Order; it is reproduced byte for byte because it is what the page posts to. |
| `/shop/payment` | page | public, site-scoped | The payment step | Hides the payment form when a blocking error exists. |
| `/shop/payment/transaction/<order>` | data | public, site-scoped | Creates the payment transaction | The access token plus the provider arguments. The refusals are WS-407 to WS-417. |
| `/shop/payment/validate` | page | public, site-scoped | Return from the provider | Redirects to the confirmation page, or back to the shop. |
| `/shop/confirmation` | page | public, site-scoped | The confirmation page | Reads the session's last order identifier. |
| `/shop/print` | page | public, site-scoped | The printable order document | The rendered order report, or a redirect to the shop. |
| `/shop/express_checkout` | data, submission request | public, site-scoped | Records the wallet addresses and option | Returns the order's customer identifier. |
| `/shop/express/shipping_address_change` | data | public, site-scoped | Delivery options for a partial address | The delivery methods, sorted by increasing price. |
| `/shop/express/shipping_address_change/compute_taxes` | data | public, site-scoped | Recomputes taxes in express mode | The total excluding delivery in minor currency units, or the external tax error flag. |

### 3.5 Delivery and collection

| Path | Kind | Access | Purpose | Response |
|---|---|---|---|---|
| `/shop/delivery_methods` | data | public, site-scoped | Renders the delivery block | The rendered block, built from the available methods, the selected one, the order and, with collection in store, the default pickup locations. |
| `/shop/set_delivery_method` | data | public, site-scoped | Selects a method | The order summary values; refuses a change once a transaction exists. |
| `/shop/get_delivery_rate` | data, submission request | public, site-scoped | Rates one method without selecting it | The rate payload with the formatted amount, the free-delivery flag and the invoice-after-shipping flag. |
| `/website_sale/get_pickup_locations` | data | public, site-scoped | The pickup points near a postal code | The pickup locations, or the error `No pick-up points are available for this delivery address.` |
| `/website_sale/set_pickup_location` | data | public, site-scoped | Stores the chosen pickup point | Nothing. |
| `/shop/set_click_and_collect_location` | data | public, site-scoped | Chooses a store from the product page | Nothing; creates the cart and the collect-in-store delivery line when needed. |
| `/website_sale_mondialrelay/update_shipping` | data | public, site-scoped | Stores the chosen external relay point | The re-rendered address block and the new delivery contact. |

### 3.6 Engagement

| Path | Kind | Access | Purpose |
|---|---|---|---|
| `/shop/wishlist/add` | data | public, site-scoped | Saves a product with the price and price list of the moment. |
| `/shop/wishlist` | page | public, site-scoped | The wish list page. |
| `/shop/wishlist/remove/<row>` | data | public, site-scoped | Removes a saved product. |
| `/shop/wishlist/get_product_ids` | data, read only | public, site-scoped | The identifiers of the saved variants. |
| `/shop/compare` | page | public, site-scoped | The comparison page. |
| `/shop/compare/get_product_data` | data | public, site-scoped | The comparison columns payload. |
| `/shop/add/stock_notification` | data | public, site-scoped | Subscribes to a back-in-stock notification. |
| `/coupon/<code>` | page | public, site-scoped | Remembers or applies a coupon code. |
| `/shop/claimreward` | page | public, site-scoped | Claims a reward. |
| `/wallet/top_up` | page | signed in, site-scoped | Adds the electronic wallet's trigger product to the cart. |
| `/donation/pay` | page, retrieval and submission request | public, site-scoped | The donation payment page. |
| `/donation/transaction/<minimum amount>` | data | public, site-scoped | Creates the donation transaction. |
| `/website_payment/snippet/supported_payment_methods` | page, retrieval request, read only | public, site-scoped | The payment methods advertised on the site. |
| `/slides/get_course_products` | data | signed in | The course products with their formatted price. |

### 3.7 Storefront editing

| Path | Access | Writes |
|---|---|---|
| `/shop/product/extra-media` | Restricted Editor | New Product Image rows, pictures or one video, on a variant or a template. |
| `/shop/product/clear-images` | Restricted Editor | Deletes the variant media, or the template media when the variant has none. |
| `/shop/product/resequence-image` | Restricted Editor | The media order, with the main-picture swap rules. |
| `/shop/config/product` | Restricted Editor | The shop ordering value or the tile size of a product. |
| `/shop/config/attribute` | Restricted Editor | The display type of an attribute; clears the page template cache. |
| `/shop/config/website` | Restricted Editor | The allow-listed page style settings, the wish list settings and the extra-step toggle. |
| `/shop/config/category` | write access to the category | The three category page options. |
| `/snippets/category/set_image` | Restricted Editor | The cover picture of a category. |

---

## 4. Generated documents

### 4.1 The crawler exclusion response

**Trigger.** A request on the crawler exclusion path. **Format.** Plain text, never language-prefixed.
**Content.** The user-agent line, one allow line per endpoint declared as allowed, then either a
disallow-everything block pointing at the canonical host's index, or the index of the request root followed
by the custom block under its banner (§14 of [workflows.md](workflows.md)).

### 4.2 The site index

**Trigger.** A request on the site index path. **Format.** The extensible markup language, with the
character set declared. **Content.** One location per enumerated address, with an optional last
modification date and an optional priority. **Chunking.** At most 45000 locations per document; more
produce an index of indexes. **Caching.** Twelve hours, per site and per host.

### 4.3 The blog subscription feed

**Trigger.** A request on a blog's feed address. **Format.** The syndication document format, with its own
media type. **Content.** The most recent posts, at most 50 and 15 by default, each with its title, its
absolute address and the plain text of its content.

### 4.4 The product syndication feed

**Trigger.** A fetch of the feed path with a valid feed identifier and access token. The path is a fixed
literal with no variable part: a slash, the three lower-case letters that abbreviate the name of the Google
Merchant Center product listing service, a dot, and the three lower-case letters that abbreviate extensible
markup language. The two query parameter names are external wire literals: the external service stores the
whole address and replays it unchanged, so a rebuild must answer that exact path with those exact parameter
names.

**Format.** A channel document with a title, a link, a description and one item per product. The complete
field list is in [storefront-catalogue.md](storefront-catalogue.md) §16.3.

**Grouping and totals.** None; the document is a flat list. Variants of one template are linked by a shared
group identifier.

**Transport.** The response declares the extensible markup language content type with the character set,
and a compressed content encoding. The body is the compressed document.

**Caching.** One rendering per feed per day, under an exclusive row lock; parameter changes invalidate the
cache immediately.

### 4.5 Printed documents

| Document | Trigger | Content |
|---|---|---|
| Order document | The print address after a checkout, and the portal order page | The standard order report of the [sales](../sales/README.md) folder, rendered for the session's last order. |
| Invoice document | The portal invoice page | Owned by [accounts receivable](../accounts-receivable/README.md). This folder only makes the provider filtering on that page site-aware. |

This folder adds no report of its own.

---

## 5. Notifications and messages

| Message | Trigger | Recipient | Content |
|---|---|---|---|
| Cart recovery | The hourly job, or the manual action | The cart's customer | Subject `You left items in your cart!`; the cart content; a button labelled `Resume Order` pointing at the revival address. |
| Back in stock | The hourly job | Every subscriber of a product that is no longer sold out | Subject `The product '%(product_name)s' is now available`, in the subscriber's language; sender the company contact, else the site salesperson. |
| Order confirmation | Order confirmation | The customer | The site's confirmation template when set, otherwise the platform default. |
| Payment status | A transaction reaching the pending state | The customer | The standard payment-status message; for a wire transfer it carries the transfer instructions. |
| Salesperson assignment | Order confirmation, and before the payment-succeeded message | The assigned salesperson | The standard assignment notification, authored by the platform system user rather than by the shopper. |
| Feed product limit | A feed rendering above the warning threshold, at most once a week | The site salesperson | Subject `GMC: Product Limit Exceeded`, reproduced verbatim; body `The feed %(feed_name)s contains more than %(limit)s products, which may not be fully updated. Consider refining the feed by adjusting the product categories.` |
| Donation confirmation | A donation transaction reaching the completed state | The donor | Subject `Donation confirmation`. |
| Donation notice | A donation transaction is created | The address supplied as the donation recipient | Subject `A donation has been made on your website`, with the donor's comment. |
| Open cart price list warning | Changing the assigned price list of a contact who has an open storefront cart | The user making the change, as a form warning | Title `Open Sale Orders`; body `This partner has an open cart. Please note that the pricelist will not be updated on that cart. Also, the cart might not be visible for the customer until you update the pricelist of that cart.` |
| New blog post | Publishing a post | The blog's followers | The shipped new-post template, with the post title as subject. |
| New forum question, new forum answer | A post becomes active | The followers of the post and of its tags | The shipped templates; the answer subject is `Re: ` followed by the question title. |
| Forum validation request | A question is created pending | Every moderator and tag follower, as an internal note | The shipped validation template. |
| Public form notification | A submission on a form whose target is the outgoing mail entity | The recipient named in the form | The assembled message of §27 of [workflows.md](workflows.md), sent immediately. |
| Profile address validation | The account asks for validation | The account | A link carrying the one-day token of WS-560. |

---

## 6. Scheduled jobs

Listed with their interval and their idempotence in [configuration.md](configuration.md) §6.

---

## 7. Storefront screens

### 7.1 The shop listing

**Layout.** A header with the search box, the price list selector, the wish list counter and the cart
counter; an optional category strip or side category tree; an optional filter panel with attributes, tags
and a price slider; a toolbar with the sort selector and the grid or list switch; the product grid; the
pager.

**Per tile.** The product picture, at 512 or 1024 pixels depending on the tile size and the number of
columns; the ribbon; the name; the optional description; the price with its optional strikethrough; the
optional per-unit price; the previewed attribute values; and the actions: add to cart, add to the wish
list, add to the comparison. The action placement — inline, on hover, promoted — is a presentation token of
the site.

| Button | Guard |
|---|---|
| Add to cart, quick add | The product matches the site product condition; the price is not zero when the site forbids zero prices; the product is not sold out. |
| Add to the wish list | Shown when the wish list option is active; an anonymous visitor's rows live in the session. |
| Add to the comparison | Shown when the comparison option is active. |
| Sort selector | Always. |
| Grid or list switch | Shown when the corresponding page option is active. |
| Price list selector | Shown when at least one selectable price list exists. |

**Filters.** Category as a path segment; attribute values as a repeated query parameter; tags as a
comma-separated query parameter; a price range as two query parameters; a search term.
**Groupings.** None; the listing is flat and ordered.
**Editor controls.** Products per page, columns, gap, container width, default sort, tile design, product
order (top, bottom, up, down), tile size and the category page options.

### 7.2 The product page

**Layout.** Breadcrumb; media column, a carousel or a grid; details column with the name, the storefront
description, the price block, the tax indication, the attribute controls, the quantity and the actions;
informative attributes; specification table; documents; extra fields; tags; reviews; alternative products;
cross-selling blocks.

**Status indicators.** The publication toggle for an editor; the availability line; the ribbon.

| Button | Guard |
|---|---|
| Add to cart | The add-to-cart permission check; the configurator dialogue opens first when required. |
| Buy now | Shown when that page option is active; it adds and goes straight to the cart summary step. |
| Contact us | Replaces the add-to-cart button when the price is hidden. |
| Notify me when back in stock | Shown when the product is sold out and tracked. |
| Choose a store | Shown when collection in store is enabled and the collect-in-store method has more than one store. |
| Add to the wish list, add to the comparison | The corresponding page options. |

### 7.3 The cart page

**Content.** The cart lines with picture, header, combination name, description, unit price, quantity
input, line total and a remove action; the accessory suggestions; the quick reorder history grouped by day
label; the promotional code box; the totals block with the untaxed amount, the taxes, the delivery amount
and the total; the express checkout buttons; the checkout button labelled by the next checkout step.

**Guards.** The checkout button is disabled when the cart is not ready. A zero-priced line shows its
warning. A line whose product is unpublished is not clickable.

### 7.4 The address form

**Fields.** Name, electronic mail address, telephone, company name, street, second street line, city,
state, postal code, country and, when the business-to-business block is active, the tax identification
number and the company fields. A newsletter box when that option is active.

**Behaviour.** Changing the country re-reads the mandatory field set of that country and re-labels the
state and postal code inputs. The address autocompletion service proposes completions as the shopper
types. Submission highlights the invalid inputs and prints the messages.

### 7.5 The delivery block

Each method shows its name, its storefront description, its computed price or the word for free, and, for
a method that uses locations, a store or relay selector. Selecting a method recomputes the totals. A
method whose rate fails is shown with its error message.

### 7.6 The payment step

The compatible providers and their payment methods; the stored tokens of the signed-in shopper; the terms
box when that option is active; the order summary; and the submit button labelled `Pay now`. When a
blocking error exists, the payment form is replaced by the error title and message.

### 7.7 The confirmation page

The order reference, the order summary, the payment status message, the printable document link and, for
course purchases, the links to the purchased courses.

### 7.8 The wish list and comparison pages

The wish list page lists the saved products with their saved price, their current price, an add-to-cart
action, a remove action and, with the stock capability, a back-in-stock toggle. The comparison page shows
one column per product and one row per attribute, grouped into attribute-category sections, with the price,
the strikethrough price and the media at the top of each column.

---

## 8. Site screens

### 8.1 The site dashboard

The visitor count, the page views, the top pages, the top referrers and, with the storefront, the online
sales figures over the chosen period. Opening the back-office dashboard as a user holding the sales user
group redirects here.

### 8.2 The page manager

**List.** Name, address, published, indexed, visibility, site, last content change. **Filters.**
Published, not published, indexed, my pages, site. **Groupings.** Site, visibility. **Buttons.** New Page,
Clone, Properties, Delete, Optimise search engine metadata. The list merges the Website Pages with the
Website Technical Pages, the latter being read only.

### 8.3 The menu editor

A tree of the site's entries with drag ordering, at most two levels, a mega-menu switch per entry, an
address field, an open-in-new-tab switch and a group restriction. Saving posts the full payload (§8 of
[workflows.md](workflows.md)).

### 8.4 The redirect list

**List.** Action type, name, source address, target address, site, active. **Filters.** By action type,
by site. **Form.** The endpoint helper that fills both addresses, and the validations WS-110 to WS-118.

### 8.5 The visitor list

**List.** Visitor, contact, country, language, number of visits, tracked page views, last connection,
connected. **Filters.** Connected, identified, anonymous, my site, this week. **Groupings.** Country,
language, site, contact. **Buttons.** Send a message, which requires a contact with an address (WS-244);
Product Views History, which opens the visit entries carrying a product.

### 8.6 Blog and forum back-office screens

The blog list and form, the post list, card and form with the cover editor and the publication toggle; the
forum list and form with the complete reputation table; the post list with the state filter and the
moderation actions; the tag lists.

### 8.7 Back-office storefront screens

**Online orders.** List with the reference, the creation date, the customer, the salesperson, the sales
team, the total and the state. Filters: my orders, confirmed, unpaid, abandoned, order date, from the
site, this week, this month, this year. Groupings: the standard sales groupings plus the site. The form
adds a recovery button for abandoned carts and the site field. The unpaid list is restricted to the sent
state with a site, creation disabled. The abandoned cart list is restricted to the abandoned-cart flag,
with filters on the creation date, on whether a recovery message was sent, and on the period, creation
disabled.

**Storefront product configuration.** The product form's storefront section: publication toggle, site,
storefront categories, ribbon, storefront description, extra media, alternative products, accessory
products, optional products, comparison price, base unit count and reference unit, out-of-stock policy,
availability threshold, availability display and out-of-stock message. Plus the product pages list and
card view, the Product Image form and card view, the Product Ribbon list and form with a live preview, the
Website Product Category list and form, the Product Attribute Category list, the Base Unit list, and the
Product Feed list, form and search with its cache controls.

**Reporting.** Online Sales Analysis as a pivot and a graph over the sales analysis report restricted to
rows with a site, with the confirmed filter preselected; measures: quantity, untaxed amount, total, margin
and the standard sales measures; groupings: the standard sales groupings plus the site, the storefront
categories and the abandoned-cart flag. The storefront spreadsheet dashboard. The sales team's
abandoned-cart count and amount with an action opening those carts. The periodic digest's online sales
measure, linking to the site dashboard.

---

## 9. The session payload delivered to a page

Every page receives the site identifier, the language, the front-end language list with the alternate
codes, the consent state and, on the storefront, the site's add-to-cart action and the cart quantity. The
session keys themselves are listed in [entities.md](entities.md) §5.1.

---

## 10. External integrations

| Service | Direction | Contract |
|---|---|---|
| Score-based human verification (Google reCAPTCHA) | The system calls the service | The submission token, the private key and the network address are sent to the verification endpoint with a two-second timeout; the answer carries a success flag, a score and an action name. The outcomes and the messages are in [content-management.md](content-management.md) §9.1. |
| Challenge-widget human verification (Cloudflare Turnstile) | The system calls the service | The same shape with its own token parameter, secret parameter and endpoint; the outcomes and the messages are in [content-management.md](content-management.md) §9.2. |
| External picture library | The system calls the service | A search request with the access key returns picture descriptors; the chosen picture is downloaded and stored as an attachment of the site, and a download notification is sent back to the service, which its terms require. |
| Content suggestion and text generation service | The system calls the service | The configurator asks for the industry list, for recommended themes, for industry pictures and for generated placeholder texts, sending the language name, the industry and the database identifier. Every failure is caught and leaves the shipped content in place (WS-519). |
| Mapping service | The browser and the system call the service | Static map pictures and interactive maps are built from the site's mapping access key; the map frame of the reference directory renders the published contacts. |
| Audience measurement services | The browser calls the services | The measurement script of the first service is injected with the four consent categories denied until consent is given; the second service is embedded as a dashboard with its shared key. |
| Address autocompletion service | The browser calls the service | The checkout address form proposes completions using the site's address service access key. |
| Product syndication service | The service calls the system | It fetches the feed address, with the feed identifier and the access token, at most once a day per feed (§4.4). |
| Search engine verification | The service calls the system | It fetches the verification path and expects the stored token in the body (WS-156). |
| External parcel-shop network | The browser and the system call the service | The relay-point selector returns a relay payload that becomes a delivery contact ([storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §7). |
| Print-on-demand fulfilment service | Both directions | Print pictures are synchronised onto the products; the rules are [storefront-engagement.md](storefront-engagement.md) §9. |
| Payment providers | Both directions | Owned by [payment providers](../payment-providers/README.md); this folder only restricts the compatible set per site and supplies the transaction, landing and express addresses. |

Every address of a third-party service is part of the integration contract and is reproduced where the
integration is specified; addresses that the system's own installation serves are built from the site
domain or from the platform base address.

---

## 11. Import and export

| Flow | Direction | Notes |
|---|---|---|
| Product syndication feed | Export | §4.4. The only bulk export this folder defines. |
| Site index and crawler exclusion | Export | §4.1 and §4.2; generated documents rather than data exports. |
| Blog subscription feed | Export | §4.3. |
| Theme records | Import | A theme package ships templates, assets, attachments, pages and menus; installing it copies them into live records per site (§22 of [workflows.md](workflows.md)). |
| Page and template records | Import and export | Pages, templates, menus and rewrite rules are ordinary records and use the platform's record import and export, keyed on the template key for templates and on the address for pages. A rebuild must keep the template key stable, because it is what pairs a shared template with its per-site copy. |
| Visitor and tracking data | Export | Ordinary record export. A rebuild should also offer the per-visitor erasure of WS-904. |

---

## Reconciliation notes

1. **Two endpoint catalogues.** The storefront catalogue was complete; the site catalogue existed only as
   scattered references. §2 is written here for the first time from the routing table, and §3 keeps the
   storefront catalogue unchanged apart from the renaming of the topic files it cites.
2. **Address spelling.** One version wrote endpoint paths with a placeholder name copied from the routing
   table, for example a converter with its entity name. Paths here keep the literal segments exactly and
   describe each placeholder in words, because only the literal part is contractual.
3. **The extra-information form.** Both versions reproduce the same form path. It is kept byte for byte in
   §3.4, because the page posts to it, and its last segment is documented as the transport name of the
   target entity rather than silently renamed.
