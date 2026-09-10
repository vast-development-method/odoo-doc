# Website (`website`)

**Transport name:** `website`  
**Storage name:** `website`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`  
**Extended by packages:** `website_profile`, `website_slides`, `website_sale`, `l10n_ar_website_sale`, `l10n_br_website_sale`, `website_event`, `website_event_track`, `website_blog`, `website_crm`, `website_livechat`, `website_crm_partner_assign`, `website_customer`, `website_event_exhibitor`, `website_forum`, `website_hr_recruitment`, `website_sale_autocomplete`, `website_sale_stock`, `website_sale_collect`, `website_sale_wishlist`, `website_sale_mass_mailing`

Description: Website

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (93)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Website Name | single line text |  | required |
| `sequence` | Sequence | integer |  | default `10` |
| `domain` | Website Domain | single line text |  | Help: E.g. https://www.mydomain.com |
| `domain_punycode` | Punycode Domain | single line text |  | read only; computed by rule `_compute_domain_punycode` (not stored) |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `language_ids` | Languages | many to many | `res.lang` | required; default computed dynamically (_active_languages); association table `website_lang_rel` |
| `language_count` | Number of languages | integer |  | computed by rule `_compute_language_count` (not stored) |
| `default_lang_id` | Default Language | many to one | `res.lang` | required; default computed dynamically (_default_language) |
| `auto_redirect_lang` | Autoredirect Language | boolean |  | default `True`; Help: Should users be redirected to their browser's language |
| `cookies_bar` | Cookies Bar | boolean |  | Help: Display a customizable cookies bar on your website. |
| `configurator_done` | Configurator Done | boolean |  | Help: True if configurator has been completed or ignored |
| `block_third_party_domains` | Block 3rd-party domains | boolean |  | default `True`; Help: Block 3rd-party domains that may track users (YouTube, Google Maps, etc.). |
| `custom_blocked_third_party_domains` | User list of blocked 3rd-party domains | multi line text |  | visible only to groups `website.group_website_designer` |
| `blocked_third_party_domains` | List of blocked 3rd-party domains | multi line text |  | computed by rule `_compute_blocked_third_party_domains` (not stored) |
| `logo` | Website Logo | binary |  | default computed dynamically (_default_logo); Help: Display this logo on the website. |
| `social_twitter` | X Account | single line text |  | default computed dynamically (_default_social_twitter) |
| `social_facebook` | Facebook Account | single line text |  | default computed dynamically (_default_social_facebook) |
| `social_github` | GitHub Account | single line text |  | default computed dynamically (_default_social_github) |
| `social_linkedin` | LinkedIn Account | single line text |  | default computed dynamically (_default_social_linkedin) |
| `social_youtube` | Youtube Account | single line text |  | default computed dynamically (_default_social_youtube) |
| `social_instagram` | Instagram Account | single line text |  | default computed dynamically (_default_social_instagram) |
| `social_tiktok` | TikTok Account | single line text |  | default computed dynamically (_default_social_tiktok) |
| `social_discord` | Discord Account | single line text |  | default computed dynamically (_default_social_discord) |
| `social_default_image` | Default Social Share Image | binary |  | Help: If set, replaces the website logo as the default social share image. |
| `has_social_default_image` | Has Social Default Image | boolean |  | computed by rule `_compute_has_social_default_image` and stored |
| `google_analytics_key` | Google Analytics Key | single line text |  |  |
| `google_search_console` | Google Search Console | single line text |  | Help: Google key, or Enable to access first reply |
| `google_maps_api_key` | Google Maps application programming interface Key | single line text |  |  |
| `plausible_shared_key` | Plausible Shared Key | single line text |  |  |
| `plausible_site` | Plausible Site | single line text |  |  |
| `user_id` | Public User | many to one | `res.users` | required |
| `cdn_activated` | Content Delivery Network (CDN) | boolean |  |  |
| `cdn_url` | CDN Base uniform resource locator | single line text |  | default  |
| `cdn_filters` | CDN Filters | multi line text |  | default computed dynamically (lambda s: '\n'.join(DEFAULT_CDN_FILTERS)); Help: URL matching those filters will be rewritten using the CDN Base URL |
| `partner_id` | Public Partner | many to one |  | related through path `user_id.partner_id` |
| `menu_id` | Main Menu | many to one | `website.menu` | computed by rule `_compute_menu` (not stored) |
| `homepage_url` | Homepage Uniform resource locator | single line text |  | Help: E.g. /contactus or /shop |
| `custom_code_head` | Custom <head> code | rich text |  |  |
| `custom_code_footer` | Custom end of <body> code | rich text |  |  |
| `robots_txt` | Robots.txt | rich text |  | visible only to groups `website.group_website_designer` |
| `favicon` | Website Favicon | binary |  | default computed dynamically (_default_favicon); Help: This field holds the image used to display a favicon on the website. |
| `theme_id` | Theme | many to one | `ir.module.module` | Help: Installed theme |
| `specific_user_account` | Specific User Account | boolean |  | Help: If True, new accounts will be associated to the current website |
| `auth_signup_uninvited` | Customer Account | selection |  | default `b2c`; extended by packages `website_sale` |
| `karma_profile_min` | Minimal karma to see other user's profile | integer |  | default `150` |
| `website_slide_google_app_key` | Google Doc Key | single line text |  | visible only to groups `base.group_system` |
| `salesperson_id` | Salesperson | many to one | `res.users` | restricted by domain `[["share", "=", false]]` |
| `salesteam_id` | Sales Team | many to one | `crm.team` | default computed dynamically (_default_salesteam_id); indexed (btree_not_null); on delete of the target: set null |
| `show_line_subtotals_tax_selection` | Line Subtotals Tax Display | selection |  | computed by rule `_compute_show_line_subtotals_tax_selection` and stored |
| `add_to_cart_action` | Add To Cart Action | selection |  | default `stay` |
| `account_on_checkout` | Customer Accounts | selection |  | default `optional` |
| `cart_recovery_mail_template_id` | Cart Recovery Email | many to one | `mail.template` | default computed dynamically (_default_recovery_mail_template); restricted by domain `[["model", "=", "sale.order"]]` |
| `contact_us_button_url` | Contact Us Button uniform resource locator | single line text |  | default `/contactus`; translatable |
| `cart_abandoned_delay` | Abandoned Delay | float |  | default `10.0` |
| `send_abandoned_cart_email` | Send email to customers who abandoned their cart. | boolean |  |  |
| `send_abandoned_cart_email_activation_time` | Time when the 'Send abandoned cart email' feature was activated. | date and time |  | computed by rule `_compute_send_abandoned_cart_email_activation_time` and stored |
| `shop_page_container` | Shop Page Container | selection |  | default `regular` |
| `shop_ppg` | Number of products in the grid on the shop | integer |  | default `21` |
| `shop_ppr` | Number of grid columns on the shop | integer |  | default `3` |
| `shop_gap` | Grid-gap on the shop | single line text |  | default `16px` |
| `shop_opt_products_design_classes` | Shop Design Class | single line text |  | default `o_wsale_products_opt_layout_catalog o_wsale_products_opt_design_thumbs o_wsale_products_opt_name_color_regular o_wsale_products_opt_rounded_2 o_wsale_products_opt_thumb_cover o_wsale_products_opt_img_secondary_show o_wsale_products_opt_img_hover_zoom_out_light o_wsale_products_opt_has_cta o_wsale_products_opt_actions_onhover o_wsale_products_opt_has_wishlist o_wsale_products_opt_wishlist_fixed o_wsale_products_opt_has_description o_wsale_products_opt_actions_subtle o_wsale_products_opt_cc1`; Help: CSS class for shop products design |
| `shop_default_sort` | Shop Default Sort | selection |  | required; default `website_sequence asc`; values provided by rule `_get_product_sort_mapping` |
| `shop_extra_field_ids` | E-Commerce Extra Fields | one to many | `website.sale.extra.field` | inverse field `website_id` |
| `product_page_container` | Product Page Container | selection |  | default `unset` |
| `product_page_cols_order` | Product Page main columns order | selection |  | default `regular` |
| `product_page_image_layout` | Product Page Image Layout | selection |  | required; default `carousel` |
| `product_page_image_width` | Product Page Image Width | selection |  | required; default `50_pc` |
| `product_page_image_spacing` | Product Page Image Spacing | selection |  | required; default `none` |
| `product_page_image_roundness` | Product Page Image Roundness | selection |  | required; default `none` |
| `product_page_image_ratio` | Product Page Image Ratio | selection |  | required; default `1_1` |
| `product_page_image_ratio_mobile` | Product Page Image Ratio Mobile | selection |  | required; default `auto` |
| `ecommerce_access` | Ecommerce Access | selection |  | required; default `everyone` |
| `product_page_grid_columns` | Product Page Grid Columns | integer |  | default `2` |
| `prevent_zero_price_sale` | Hide 'Add To Cart' when price = 0 | boolean |  |  |
| `enabled_gmc_src` | Google Merchant Center | boolean |  | default computed dynamically (lambda self: self.env['res.groups']._is_feature_enabled('website_sale.group_product_feed')) |
| `currency_id` | Default Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` (not stored) |
| `pricelist_ids` | Price list available for this Ecommerce/Website | one to many | `product.pricelist` | computed by rule `_compute_pricelist_ids` (not stored) |
| `confirmation_email_template_id` | Confirmation Email Template | many to one | `mail.template` | default computed dynamically (_default_confirmation_email_template); restricted by domain `[["model", "=", "sale.order"]]` |
| `l10n_ar_website_sale_show_both_prices` | Display Price without National Taxes | boolean |  | computed by rule `_compute_l10n_ar_website_sale_show_both_prices` and stored |
| `app_icon` | Website App Icon | image |  | read only; computed by rule `_compute_app_icon` and stored; Help: This field holds the image used as mobile app icon on the website (PNG format). |
| `events_app_name` | Events App Name | single line text |  | computed by rule `_compute_events_app_name` and stored; Help: This fields holds the Event's Progressive Web App name. |
| `crm_default_team_id` | Default Sales Teams | many to one | `crm.team` | restricted by domain `lambda self: self._get_crm_default_team_domain()`; Help: Default Sales Team for new leads created through the Contact Us form. |
| `crm_default_user_id` | Default Salesperson | many to one | `res.users` | restricted by domain `[["share", "=", false]]`; Help: Default salesperson for new leads created through the Contact Us form. |
| `channel_id` | Website Live Chat Channel | many to one | `im_livechat.channel` |  |
| `forum_count` | Forum Count | integer |  | read only; default  |
| `google_places_api_key` | Google Places application programming interface Key | single line text |  | visible only to groups `base.group_system` |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` |  |
| `in_store_dm_id` | In-store Delivery Method | many to one | `delivery.carrier` | computed by rule `_compute_in_store_dm_id` (not stored) |
| `wishlist_opt_products_design_classes` | Wishlist Page Design Class | single line text |  | default `o_wsale_products_opt_layout_catalog o_wsale_products_opt_design_thumbs o_wsale_products_opt_name_color_regular o_wsale_products_opt_thumb_cover o_wsale_products_opt_img_secondary_show o_wsale_products_opt_img_hover_zoom_out_light o_wsale_products_opt_has_cta o_wsale_products_opt_actions_inline o_wsale_products_opt_has_description o_wsale_products_opt_actions_promote o_wsale_products_opt_cc1`; Help: CSS class for wishlist page design |
| `wishlist_grid_columns` | Wishlist Grid Columns | integer |  | default `5`; Help: Number of columns to display on the wishlist page |
| `wishlist_mobile_columns` | Wishlist Mobile Columns | integer |  | default `2`; Help: Number of columns to display on mobile for the wishlist page (1 or 2) |
| `wishlist_gap` | Wishlist Grid Gap | single line text |  | default `16px`; Help: Gap between products on the wishlist page |
| `newsletter_id` | Newsletter List | many to one | `mailing.list` |  |

## Selection values

### `auth_signup_uninvited` (Customer Account)

| Value | Label |
|---|---|
| `b2b` | On invitation |
| `b2c` | Free sign up |

### `show_line_subtotals_tax_selection` (Line Subtotals Tax Display)

| Value | Label |
|---|---|
| `tax_excluded` | Tax Excluded |
| `tax_included` | Tax Included |

### `add_to_cart_action` (Add To Cart Action)

| Value | Label |
|---|---|
| `stay` | Stay on Product Page |
| `go_to_cart` | Go to cart |

### `account_on_checkout` (Customer Accounts)

| Value | Label |
|---|---|
| `optional` | Optional |
| `disabled` | Disabled (buy as guest) |
| `mandatory` | Mandatory (no guest checkout) |

### `shop_page_container` (Shop Page Container)

| Value | Label |
|---|---|
| `regular` | Regular |
| `fluid` | Full-width |

### `product_page_container` (Product Page Container)

| Value | Label |
|---|---|
| `unset` | Unset |
| `regular` | Regular |
| `fluid` | Full-width |

### `product_page_cols_order` (Product Page main columns order)

| Value | Label |
|---|---|
| `regular` | Regular order |
| `inverse` | Inverse order |

### `product_page_image_layout` (Product Page Image Layout)

| Value | Label |
|---|---|
| `carousel` | Carousel |
| `grid` | Grid |

### `product_page_image_width` (Product Page Image Width)

| Value | Label |
|---|---|
| `none` | Hidden |
| `33_pc` | 33 % |
| `50_pc` | 50 % |
| `66_pc` | 66 % |
| `100_pc` | 100 % |

### `product_page_image_spacing` (Product Page Image Spacing)

| Value | Label |
|---|---|
| `none` | None |
| `small` | Small |
| `medium` | Medium |
| `big` | Big |

### `product_page_image_roundness` (Product Page Image Roundness)

| Value | Label |
|---|---|
| `none` | None |
| `small` | Small |
| `medium` | Medium |
| `big` | Big |

### `product_page_image_ratio` (Product Page Image Ratio)

| Value | Label |
|---|---|
| `auto` | Auto |
| `21_9` | Wider (21/9) |
| `16_9` | Wide (16/9) |
| `4_3` | Landscape (4/3) |
| `6_5` | Horizontal (6/5) |
| `1_1` | Default (1/1) |
| `4_5` | Portrait (4/5) |
| `2_3` | Vertical (2/3) |

### `product_page_image_ratio_mobile` (Product Page Image Ratio Mobile)

| Value | Label |
|---|---|
| `auto` | Auto |
| `21_9` | Wider (21/9) |
| `16_9` | Wide (16/9) |
| `4_3` | Landscape (4/3) |
| `6_5` | Horizontal (6/5) |
| `1_1` | Default (1/1) |
| `4_5` | Portrait (4/5) |
| `2_3` | Vertical (2/3) |

### `ecommerce_access` (Ecommerce Access)

| Value | Label |
|---|---|
| `everyone` | All users |
| `logged_in` | Logged in users |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_domain_unique` | Constraint | `unique(domain)` | Website Domain should be unique. | `website` |

## Operations (161)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_website_form_last_record` | internal rule | self | `website` |  |  |
| `website_domain` | operation | self | `website` |  |  |
| `_active_languages` | internal rule | self | `website` |  |  |
| `_default_language` | preparation rule | self | `website` |  |  |
| `_default_social_facebook` | preparation rule | self | `website` |  |  |
| `_default_social_github` | preparation rule | self | `website` |  |  |
| `_default_social_linkedin` | preparation rule | self | `website` |  |  |
| `_default_social_youtube` | preparation rule | self | `website` |  |  |
| `_default_social_instagram` | preparation rule | self | `website` |  |  |
| `_default_social_twitter` | preparation rule | self | `website` |  |  |
| `_default_social_tiktok` | preparation rule | self | `website` |  |  |
| `_default_social_discord` | preparation rule | self | `website` |  |  |
| `_default_logo` | preparation rule | self | `website` |  |  |
| `_default_favicon` | preparation rule | self | `website` |  |  |
| `_onchange_language_ids` | on change | self | `website` | onchange: `language_ids` |  |
| `_compute_domain_punycode` | computation | self | `website` | depends: `domain` | Compute the punycode (ASCII-safe) version of the domain. |
| `_compute_has_social_default_image` | computation | self | `website` | depends: `social_default_image` |  |
| `_compute_language_count` | computation | self | `website` | depends: `language_ids` |  |
| `_compute_menu` | computation | self | `website` |  |  |
| `_compute_blocked_third_party_domains` | computation | self | `website` | depends: `custom_blocked_third_party_domains` |  |
| `_get_blocked_third_party_domains_list` | preparation rule | self | `website` |  |  |
| `_get_blocked_iframe_containers_classes` | preparation rule | self | `website` |  |  |
| `is_menu_cache_disabled` | operation | self | `website` |  | Checks if the website menu contains a record like url. :return: True if the menu contains a record like url |
| `create` | lifecycle override | self, vals_list | `l10n_br_website_sale`, `website_forum`, `website_sale`, `website` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website` |  |  |
| `_handle_create_write` | internal rule | self, vals | `website` | model |  |
| `_handle_favicon` | internal rule | self, vals | `website` | model |  |
| `_handle_domain` | internal rule | self, vals | `website` | model |  |
| `_normalize_domain_url` | internal rule | self, url | `website` |  | This method: - Prefixes 'https://' if it doesn't start with 'http' - Strips any tailing '/' |
| `_handle_homepage_url` | internal rule | self, vals | `website` | model |  |
| `_check_domain` | validation | self | `website` | constrains: `domain` |  |
| `_check_homepage_url` | validation | self | `website` | constrains: `homepage_url` |  |
| `_unlink_except_default_website` | internal rule | self | `website` | ondelete |  |
| `unlink` | lifecycle override | self | `website` |  |  |
| `_remove_attachments_on_website_unlink` | internal rule | self | `website` |  |  |
| `create_and_redirect_configurator` | operation | self | `website` |  |  |
| `_idna_url` | internal rule | self, url | `website` |  |  |
| `_is_indexable_url` | internal rule | self, url | `website` |  | Returns True if the given url has to be indexed by search engines. It is considered that the website must be indexed if the domain name matches the URL. We check if they are equal while ignoring the www. and http(s). This is to index the site even if the user put the www. in the settings while he has a configuration that redirects the www. to the naked domain for example (same thing for http and https).  :param url: the url to check :return: True if the url has to be indexed, False otherwise |
| `_api_rpc` | internal rule | self, route, params, endpoint_param_name, default_endpoint, **kwargs | `website` |  |  |
| `_website_api_rpc` | internal rule | self, route, params | `website` |  |  |
| `_OLG_api_rpc` | internal rule | self, route, params | `website` |  |  |
| `get_cta_data` | operation | self, website_purpose, website_type | `website_event`, `website` |  |  |
| `_get_snippet_defaults` | preparation rule | self, snippet | `website_sale`, `website` |  | Retrieve the default configuration for a given dynamic snippet. |
| `_get_snippet_view_key` | preparation rule | self, snippet, page_code | `website` |  |  |
| `_preconfigure_snippet` | internal rule | self, snippet, el, customizations | `website` |  | Apply default configuration values to a snippet element.  This ensures that when a dynamic snippet is appended via the configurator, all of its required default classes/attributes are added to the DOM element before it is rendered. |
| `_set_background_options` | internal rule | self, el, background_options | `website` |  |  |
| `get_theme_configurator_snippets` | operation | self, theme_name | `website` | model | Prepare and return configurator_snippets by fetching theme snippets and inserting addon snippets at their intended positions. |
| `configurator_set_menu_links` | operation | self, menu_company, module_data | `website_blog`, `website_forum`, `website` |  |  |
| `configurator_get_footer_links` | operation | self | `website_forum`, `website` |  |  |
| `configurator_init` | operation | self | `website` | model |  |
| `configurator_recommended_themes` | operation | self, industry_id, palette, result_nbr_max | `website` | model |  |
| `configurator_skip` | operation | self | `website` | model |  |
| `configurator_missing_industry` | operation | self, unknown_industry | `website` | model |  |
| `configurator_apply` | operation | self, **kwargs | `website_sale`, `website` | model | Override of `website` to apply eCommerce page style configurations.  :param str shop_page_style_option: The key of the selected shop page style option. See                                    `const.SHOP_PAGE_STYLE_MAPPING`. :param str product_page_style_option: The key of the selected product page style option. See                                       `const.PRODUCT_PAGE_STYLE_MAPPING`. |
| `configurator_addons_apply` | operation | self, industry_name, **kwargs | `website_sale`, `website` |  | Override of `website` to generate eCommerce categories for a given industry using AI. |
| `_bootstrap_homepage` | internal rule | self | `website` |  |  |
| `copy_menu_hierarchy` | operation | self, top_menu | `website` |  |  |
| `new_page` | operation | self, name, add_menu, template, ispage, namespace, page_values, menu_values, sections_arch, page_title | `website_event`, `website` | model | Create a new website page, and assign it a xmlid based on the given one :param name: the name of the page :param add_menu: if True, add a menu for that page :param template: potential xml_id of the page to create :param namespace: module part of the xml_id if none, the template module name is used :param page_values: default values for the page to be created :param menu_values: default values for the menu to be created :param sections_arch: HTML content of sections :param page_title: if set, it allows using 'name' for the URL and a different title |
| `get_unique_path` | operation | self, page_url | `website` |  | Given an url, return that url suffixed by counter if it already exists :param page_url : the url to be checked for uniqueness |
| `_get_plausible_script_url` | preparation rule | self | `website` |  |  |
| `_get_plausible_server` | preparation rule | self | `website` |  |  |
| `_get_plausible_share_url` | preparation rule | self | `website` |  |  |
| `get_unique_key` | operation | self, string, template_module | `website` |  | Given a string, return an unique key including module prefix. It will be suffixed by a counter if it already exists to garantee uniqueness. :param string : the key to be checked for uniqueness, you can pass it with 'website.' or not :param template_module : the module to be prefixed on the key, if not set, we will use website |
| `search_url_dependencies` | operation | self, res_model, res_ids | `website` | model | Search dependencies just for information. It will not catch 100% of dependencies and False positive is more than possible Each module could add dependences in this dict  :returns: a dictionnary where key is the 'categorie' of object related to the given     view, and the value is the list of text and link to the resource using given page |
| `get_current_website` | operation | self, fallback | `website` | model | The current website is returned in the following order:  - the website forced in session `force_website_id` - the website set in context - (if frontend or fallback) the website matching the request's "domain" - arbitrary the first website found in the database if `fallback` is set   to `True` - empty browse record |
| `_get_current_website_id` | preparation rule | self, domain_name, fallback | `website` | model | Get the current website id.  First find the website for which the configured `domain` (after ignoring a potential scheme) is equal to the given `domain_name`. If a match is found, return it immediately.  If there is no website found for the given `domain_name`, either fallback to the first found website (no matter its `domain`) or return False depending on the `fallback` parameter.  :param domain_name: the domain for which we want the website.     In regard to the `url_parse` method, only the `netloc` part should     be given here, no `scheme`. :type domain_name: string  :param fallback: if Tr |
| `_force` | internal rule | self | `website` |  |  |
| `_force_website` | internal rule | self, website_id | `website` |  |  |
| `is_public_user` | operation | self | `website` | model |  |
| `viewref` | operation | self, view_id, raise_if_not_found | `website` | model | Given an xml_id or a view_id, return the corresponding view record. In case of website context, return the most specific one.  Look also for archived views, no matter the context.  :param view_id: either a string xml_id or an integer view_id :param raise_if_not_found: should the method raise an error if no view found :return: The view record or empty recordset |
| `is_view_active` | operation | self, key | `website` | model | Return True if active, False if not active, None if not found |
| `get_template` | operation | self, template | `website` | model |  |
| `pager` | operation | self, url, total, page, step, scope, url_args | `website` | model |  |
| `rule_is_enumerable` | operation | self, rule | `website` |  | Checks that it is possible to generate sensible GET queries for a given rule (if the endpoint matches its own requirements) :type rule: werkzeug.routing.Rule :rtype: bool |
| `_enumerate_pages` | internal rule | self, query_string, force | `website` |  | Available pages in the website/CMS. This is mostly used for links generation and can be overridden by modules setting up new HTML controllers for dynamic pages (e.g. blog). By default, returns template views marked as pages. :param str query_string: a (user-provided) string, fetches pages                          matching the string :returns: a list of mappings with two keys: ``name`` is the displayable           name of the resource (page), ``url`` is the absolute URL           of the same. :rtype: list({name: str, url: str}) |
| `get_website_page_ids` | operation | self | `website` |  | Returns website page IDs grouped by website.  If called with an empty or non-existent recordset, returns all pages under the None key. Else, returns a mapping of website IDs to their respective page IDs.  :returns: Dict mapping website ID (or None) to list of website.page IDs. :rtype: dict[int \| None, list[int]] |
| `_get_website_pages` | preparation rule | self, domain, order, limit | `website` |  |  |
| `search_pages` | operation | self, needle, limit | `website` |  |  |
| `check_existing_page` | operation | self, page | `website` |  | Returns a boolean, whether the page is considered to exist for the current website. This is a heuristic and is not perfectly reliable. |
| `get_suggested_controllers` | operation | self | `website_blog`, `website_crm_partner_assign`, `website_customer`, `website_event`, `website_forum`, `website_hr_recruitment`, `website_sale`, `website_slides`, `website` |  | Returns a tuple (name, url, icon). Where icon can be a module name, or a path |
| `image_url` | operation | self, record, field, size | `website` | model | Returns a local url that points to the image field of a given browse record. |
| `get_cdn_url` | operation | self, uri | `website` |  |  |
| `action_dashboard_redirect` | user action | self | `website_sale`, `website` | model |  |
| `get_client_action_url` | operation | self, url, mode_edit, mode_debug | `website` |  |  |
| `get_client_action` | operation | self, url, mode_edit, website_id | `website` |  |  |
| `button_go_website` | user action | self, path | `website` |  |  |
| `_get_canonical_url` | preparation rule | self | `website_sale`, `website` |  | Returns the canonical URL of the current request. |
| `_is_canonical_url` | internal rule | self | `website` |  | Returns whether the current request URL is canonical. |
| `_get_cached_values` | preparation rule | self | `website` |  |  |
| `_get_cached` | preparation rule | self, field | `website` |  |  |
| `_get_html_fields_blacklist` | preparation rule | self | `website` |  |  |
| `_get_html_fields` | preparation rule | self | `website` |  |  |
| `_is_snippet_used` | internal rule | self, snippet_module, snippet_id, asset_version, asset_type, html_fields | `website` |  |  |
| `_check_snippet_used` | validation | self, snippet_occurences, asset_type, asset_version | `website` |  |  |
| `_check_user_can_modify` | validation | self, record | `website` |  | Verify that the current user can modify the given record.  :param record: record on which to perform the check :raise AccessError: if the operation is forbidden |
| `_disable_unused_snippets_assets` | internal rule | self | `website` |  |  |
| `_search_build_domain` | search rule | self, domain_list, search, fields, extra | `website` |  | Builds a search domain AND-combining a base domain with partial matches of each term in the search expression in any of the fields.  :param domain: base domain combined in the search expression :param search: search expression string :param fields: list of field names to match the terms of the search expression with :param extra: function that returns an additional subdomain for a search term  :return: domain limited to the matches of the search expression |
| `_search_text_from_html` | search rule | self, html_fragment | `website` |  | Returns the plain non-tag text from an html  :param html_fragment: document from which text must be extracted  :return text extracted from the html |
| `_search_get_details` | search rule | self, search_type, order, options | `website_blog`, `website_event_exhibitor`, `website_event_track`, `website_event`, `website_forum`, `website_hr_recruitment`, `website_sale`, `website_slides`, `website` |  | Returns indications on how to perform the searches  :param search_type: type of search :param order: order in which the results are to be returned :param options: search options  :return: list of search details obtained from the `website.searchable.mixin`'s `_search_get_detail()` |
| `_search_with_fuzzy` | search rule | self, search_type, search, limit, order, options | `website` |  | Performs a search with a search text or with a resembling word  :param search_type: indicates what to search within, 'all' matches all available types :param search: text against which to match results :param limit: maximum number of results per model type involved in the result :param order: order on which to sort results within a model type :param options: search options from the submitted form containing:     - allowFuzzy: boolean indicating whether the fuzzy matching must be done     - other options used by `_search_get_details()`  :return: tuple containing:     - count: total number of re |
| `_search_exact` | search rule | self, search_details, search, limit, order | `website` |  | Performs a search with a search text  :param search_details: see :meth:`_search_get_details` :param search: text against which to match results :param limit: maximum number of results per model type involved in the result :param order: order on which to sort results within a model type  :return: tuple containing:     - total number of results across all involved models     - list of results per model made of:         - initial search_detail for the model         - count: number of results for the model         - results: model list equivalent to a `model.search()` |
| `_search_render_results` | search rule | self, search_details, limit | `website` |  | Prepares data for the autocomplete and hybrid list rendering  :param search_details: obtained from `_search_exact()` :param limit: maximum number or rows to render  :return: the updated `search_details` containing an additional `results_data` field equivalent     to the result of a `model.read()` |
| `_search_find_fuzzy_term` | search rule | self, search_details, search, limit, word_list | `website` |  | Returns the "closest" match of the search parameter within available words.  :param search_details: obtained from `_search_get_details()` :param search: search term to which words must be matched against :param limit: maximum number of records fetched per model to build the word list :param word_list: if specified, this list of words is used as possible targets instead of     the words contained in the match fields of each involved model  :return: term on which a search can be performed instead of the initial search |
| `_search_get_indirect_fields` | search rule | self, fields, model | `website` |  | Returns the list of indirect fields amongst the requested fields.  :param fields: list of field names to be searched :param model: model within which to search :return: dict of indirect field details per indirect field name |
| `_trigram_enumerate_words` | internal rule | self, search_details, search, limit | `website` |  | Browses through all words that need to be compared to the search term. It extracts all words of every field associated to models in the fields_per_model parameter. The search is restricted to a records having the non-zero pg_trgm.word_similarity() score.  :param search_details: obtained from `_search_get_details()` :param search: search term to which words must be matched against :param limit: maximum number of records fetched per model to build the word list :return: yields words |
| `_basic_enumerate_words` | internal rule | self, search_details, search, limit | `website` |  | Browses through all words that need to be compared to the search term. It extracts all words of every field associated to models in the fields_per_model parameter.  :param search_details: obtained from `_search_get_details()` :param search: search term to which words must be matched against :param limit: maximum number of records fetched per model to build the word list :return: yields words |
| `_allConsentsGranted` | internal rule | self | `website` |  | Checks if all (cookies) consents have been granted. Note that in the case no cookies bar has been enabled, this considers that full consent has been immediately given. Indeed, in that case, we suppose that the user implemented his own consent behavior through custom code / app. That custom code / app is able to override this function as desired and xpath the `tracking_code_config` script in `website.layout`.  :return: True if all consents have been granted, False otherwise |
| `_control_third_party_trackers_in_html` | internal rule | self, html_content | `website` |  |  |
| `_should_remove_third_party_trackers` | internal rule | self | `website` |  |  |
| `_remove_third_party_trackers` | internal rule | self, tagName, atts, cookies_watchlist | `website` |  |  |
| `_is_tag_domains_watchlisted` | internal rule | self, tagName, atts | `website` |  |  |
| `_is_tag_classes_watchlisted` | internal rule | self, tagName, atts | `website` |  |  |
| `_default_salesteam_id` | preparation rule | self | `website_sale` |  |  |
| `_default_recovery_mail_template` | preparation rule | self | `website_sale` |  |  |
| `_default_confirmation_email_template` | preparation rule | self | `website_sale` |  |  |
| `_compute_pricelist_ids` | computation | self | `website_sale` |  |  |
| `_compute_currency_id` | computation | self | `website_sale` | depends: `company_id` |  |
| `_compute_send_abandoned_cart_email_activation_time` | computation | self | `website_sale` | depends: `send_abandoned_cart_email` |  |
| `_compute_show_line_subtotals_tax_selection` | computation | self | `l10n_ar_website_sale`, `website_sale` | depends: `company_id.account_fiscal_country_id` |  |
| `_get_product_sort_mapping` | preparation rule |  | `website_sale` |  |  |
| `get_configurator_shop_page_styles` | operation | self | `website_sale` | model | Format and return the ids and images of each shop page style for website onboarding.  :return: The shop page style information. :rtype: list[dict] |
| `get_configurator_product_page_styles` | operation | self | `website_sale` | model | Format and return ids and images of each product page style for website onboarding.  :return: The product page style information. :rtype: list[dict] |
| `_get_pl_partner_order` | preparation rule | self, country_code, show_visible, current_pl_id, website_pricelist_ids, partner_pl_id | `website_sale` |  | Return the list of pricelists that can be used on website for the current user.  :param str country_code: code iso or False, If set, we search only price list available for this country :param bool show_visible: if True, we don't display pricelist where selectable is False (Eg: Code promo) :param int current_pl_id: The current pricelist used on the website     (If not selectable but currently used anyway, e.g. pricelist with promo code) :param tuple website_pricelist_ids: List of ids of pricelists available for this website :param int partner_pl_id: the partner pricelist :returns: list of prod |
| `get_pricelist_available` | operation | self, show_visible | `website_sale` |  | Return the list of pricelists that can be used on website for the current user. Country restrictions will be detected with GeoIP (if installed). :param bool show_visible: if True, we don't display pricelist where selectable is False (Eg: Code promo) :returns: pricelist recordset |
| `is_pricelist_available` | operation | self, pl_id | `website_sale` |  | Return a boolean to specify if a specific pricelist can be manually set on the website. Warning: It check only if pricelist is in the 'selectable' pricelists or the current pricelist. :param int pl_id: The pricelist id to check :returns: Boolean, True if valid / available |
| `_get_geoip_country_code` | preparation rule | self | `website_sale` |  |  |
| `sale_product_domain` | operation | self | `website_sale` |  |  |
| `_product_domain` | internal rule | self | `website_sale` |  |  |
| `_create_cart` | internal rule | self | `website_sale` |  |  |
| `_prepare_sale_order_values` | preparation rule | self, partner_sudo | `website_sale` |  |  |
| `_get_and_cache_current_pricelist` | preparation rule | self | `website_sale` |  | Retrieve and cache the current pricelist for the session.  Note: self.ensure_one()  :return: The determined pricelist, which could be empty, as a sudoed record. :rtype: product.pricelist |
| `_get_and_cache_current_fiscal_position` | preparation rule | self | `website_sale` |  | Retrieve and cache the current fiscal position for the session.  Note: self.ensure_one()  :return: A sudoed fiscal position record. :rtype: account.fiscal.position |
| `_get_and_cache_current_cart` | preparation rule | self | `website_sale` |  | Retrieves and caches the current cart for the session.  Note: self.ensure_one()  :return: A sudoed Sales order record. :rtype: sale.order |
| `sale_reset` | operation | self | `website_sale` |  |  |
| `_get_product_page_proportions` | preparation rule | self | `website_sale` |  | Returns the number of columns (css) that both the images and the product details should take. |
| `_get_product_page_grid_image_spacing_classes` | preparation rule | self | `website_sale` |  |  |
| `_get_product_page_grid_image_rounded_classes` | preparation rule | self | `website_sale` |  |  |
| `_get_product_page_container` | preparation rule | self | `website_sale` |  |  |
| `_send_abandoned_cart_email` | internal rule | self | `website_sale` | model |  |
| `_create_checkout_steps` | internal rule | self | `website_sale` |  |  |
| `_get_checkout_step` | preparation rule | self, href | `website_sale` |  |  |
| `_get_allowed_steps_domain` | preparation rule | self | `website_sale` |  |  |
| `_get_checkout_steps` | preparation rule | self | `website_sale` |  |  |
| `_get_checkout_step_values` | preparation rule | self | `website_sale` |  |  |
| `has_ecommerce_access` | operation | self | `website_sale` |  | Return whether the current user is allowed to access eCommerce-related content. |
| `_get_product_image_ratio` | preparation rule | self | `website_sale` |  | Get the product image aspect ratio based on the website's design classes.  Returns:     str: The aspect ratio as a string (e.g., '16_9', '4_3', '1_1') |
| `_get_product_image_ratio_height` | preparation rule | self | `website_sale` |  |  |
| `_get_basic_feed_product_domain` | preparation rule | self | `website_sale` |  |  |
| `_default_feed_is_valid` | preparation rule | self | `website_sale` |  |  |
| `_populate_product_feeds` | internal rule | self | `website_sale` |  | Populate product feeds for the website with default values. |
| `_compute_l10n_ar_website_sale_show_both_prices` | computation | self | `l10n_ar_website_sale` | depends: `company_id` |  |
| `_compute_events_app_name` | computation | self | `website_event_track` | depends: `name` |  |
| `_check_events_app_name` | validation | self | `website_event_track` | constrains: `events_app_name` |  |
| `_compute_app_icon` | computation | self | `website_event_track` | depends: `favicon` | Computes a squared image based on the favicon to be used as mobile webapp icon. App Icon should be in PNG format and size of at least 512x512.  If the favicon is an SVG image, it will be skipped and the app_icon will be set to False. |
| `_get_crm_default_team_domain` | preparation rule | self | `website_crm` |  |  |
| `_get_livechat_channel_info` | preparation rule | self | `website_livechat` |  | Get the livechat info dict (button text, channel name, ...) for the livechat channel of the current website. |
| `_update_forum_count` | internal rule | self | `website_forum` |  | Update count of forum linked to some websites. This has to be done manually as website_id=False on forum model means a shared forum. There is therefore no straightforward relationship to be used between forum and website.  This method either runs on self (if not void), either on all existing websites (to update globally counters, notably when a new forum is created). |
| `has_google_places_api_key` | operation | self | `website_sale_autocomplete` |  |  |
| `_get_product_available_qty` | preparation rule | self, product, **kwargs | `website_sale_collect`, `website_sale_stock` |  | Give the available quantity of a given product.  NB: this method is only meant to be used on the shop before the checkout. For checkout steps, please use `cart._get_free_qty` instead to consider the chosen warehouse for delivery (website_sale_collect).  :param product: product.product record :param dict kwargs: unused parameters, available for overrides :return: available quantity :rtype: float |
| `_compute_in_store_dm_id` | computation | self | `website_sale_collect` |  |  |
| `_get_max_in_store_product_available_qty` | preparation rule | self, product | `website_sale_collect` |  | Return maximum amount of product available to deliver with in store delivery method. |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_domain` | ValidationError | The domain path cannot contain relative path segments like '/./' or '/../'. | `website` |
| `_check_domain` | ValidationError | The provided website domain is not a valid URL. | `website` |
| `_check_homepage_url` | ValidationError | The homepage URL should be relative and start with '/'. | `website` |
| `_unlink_except_default_website` | UserError | You cannot delete default website %s. Try to change its settings instead | `website` |
| `get_website_page_ids` | AccessError | Access Denied | `website` |
| `action_dashboard_redirect` | AccessError | You don't have the necessary access rights to access this dashboard. | `website` |
| `_check_events_app_name` | ValidationError | "Events App Name" field is required. | `website_event_track` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website` |
| `base.group_portal` | no | yes | no | no | `website` |
| `base.group_user` | no | yes | no | no | `website` |
| `group_website_designer` | yes | yes | yes | yes | `website` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.view_website_form` | form |  | `name`, `domain`, `logo`, `language_count`, `company_id`, `language_ids`, `default_lang_id`, `custom_code_head`, `custom_code_footer` |  |  | `website` |
| `website.view_website_form_view_themes_modal` | xpath | `website.view_website_form` |  | `Create`, `Cancel` |  | `website` |
| `website.view_website_tree` | list |  | `sequence`, `name`, `domain`, `company_id`, `default_lang_id`, `theme_id` |  |  | `website` |
| `website_profile.website_view_form` | xpath | `website.view_website_form` | `karma_profile_min` |  |  | `website_profile` |
| `website_sale.view_website_sale_website_form` | notebook | `website.view_website_form` | `shop_extra_field_ids`, `sequence`, `field_id` |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website.action_website_list` | Websites | list,form |  |  | current | `website` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `website.ir_actions_server_website_dashboard` | Website: Dashboard | code |  | yes |
| `website.ir_actions_server_website_analytics` | Website: Analytics | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `website.website_disable_unused_snippets_assets` | Disable unused snippets assets | 1 weeks | `_disable_unused_snippets_assets` |  |
| `website_sale.ir_cron_send_availability_email` | eCommerce: send email to customers about their abandoned cart | 1 hours | `_send_abandoned_cart_email` |  |

Machine-readable definition: `../../../schemas/data/entities/website.json`; views: `../../../schemas/interfaces/views/website.json`.
