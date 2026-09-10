# Shipping Methods (`delivery.carrier`)

**Transport name:** `delivery.carrier`  
**Storage name:** `delivery_carrier`  
**Kind:** persistent entity (one table)  
**Defined by package:** `delivery`  
**Extended by packages:** `stock_delivery`, `delivery_mondialrelay`, `website_sale`, `l10n_ro_edi_stock`, `sale_gelato`, `website_sale_collect`

Description: Shipping Methods

## Identity and behavior

- Mixins (classical inheritance): `website.published.multi.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (42)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Delivery Method | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | default `10`; Help: Determine the display order |
| `delivery_type` | Provider | selection |  | required; default `fixed`; on delete of the target: {"in_store": "set default"}; extended by packages `sale_gelato`, `website_sale_collect` |
| `allow_cash_on_delivery` | Cash on Delivery | boolean |  | Help: Allow customers to choose Cash on Delivery as their payment method. |
| `integration_level` | Integration Level | selection |  | default `rate_and_ship`; Help: Action while validating Delivery Orders |
| `prod_environment` | Environment | boolean |  | Help: Set to True if your credentials are certified for production. |
| `debug_logging` | Debug logging | boolean |  | Help: Log requests in order to ease debugging |
| `company_id` | Company | many to one | `res.company` | related through path `product_id.company_id` and stored |
| `product_id` | Delivery Product | many to one | `product.product` | required; on delete of the target: restrict |
| `tracking_url` | Tracking Link | single line text |  | Help: This option adds a link for the customer in the portal to track their package easily. Use <shipmenttrackingnumber> as a placeholder in your URL. |
| `currency_id` | Currency | many to one |  | related through path `product_id.currency_id` |
| `invoice_policy` | Invoicing Policy | selection |  | required; default `estimated`; on delete of the target: {"real": "set default"}; Help: Estimated Cost: the customer will be invoiced the estimated cost of the shipping. Real Cost: the customer will be invoiced the real cost of the shipping, the cost of theshipping will be updated on the SO after the delivery.; extended by packages `stock_delivery` |
| `country_ids` | Countries | many to many | `res.country` | association table `delivery_carrier_country_rel` |
| `state_ids` | States | many to many | `res.country.state` | association table `delivery_carrier_state_rel` |
| `zip_prefix_ids` | Zip Prefixes | many to many | `delivery.zip.prefix` | association table `delivery_zip_prefix_rel`; Help: Prefixes of zip codes that this carrier applies to. Note that regular expressions can be used to support countries with varying zip code lengths, i.e. '$' can be added to end of prefix to match the exact zip (e.g. '100$' will only match '100' and not '1000') |
| `max_weight` | Max Weight | float |  | Help: If the total weight of the order is over this weight, the method won't be available. |
| `weight_uom_name` | Weight unit of measure label | single line text |  | computed by rule `_compute_weight_uom_name` (not stored) |
| `max_volume` | Max Volume | float |  | Help: If the total volume of the order is over this volume, the method won't be available. |
| `volume_uom_name` | Volume unit of measure label | single line text |  | computed by rule `_compute_volume_uom_name` (not stored) |
| `must_have_tag_ids` | Must Have Tags | many to many | `product.tag` | association table `product_tag_delivery_carrier_must_have_rel`; Help: The method is available only if at least one product of the order has one of these tags. |
| `excluded_tag_ids` | Excluded Tags | many to many | `product.tag` | association table `product_tag_delivery_carrier_excluded_rel`; Help: The method is NOT available if at least one product of the order has one of these tags. |
| `carrier_description` | Carrier Description | multi line text |  | translatable; Help: A description of the delivery method that you want to communicate to your customers on the Sales Order and sales confirmation email.E.g. instructions for customers to follow. |
| `margin` | Margin | float |  | Help: This percentage will be added to the shipping price. |
| `fixed_margin` | Fixed Margin | float |  | Help: This fixed amount will be added to the shipping price. |
| `free_over` | Free if order amount is above | boolean |  | default ; Help: If the order total amount (shipping excluded) is above or equal to this value, the customer benefits from a free shipping |
| `amount` | Amount | float |  | default `1000`; Help: Amount of the order to benefit from a free shipping, expressed in the company currency |
| `can_generate_return` | Can Generate Return | boolean |  | computed by rule `_compute_can_generate_return` (not stored) |
| `return_label_on_delivery` | Generate Return Label | boolean |  | Help: The return label is automatically generated at the delivery. |
| `get_return_label_from_portal` | Return Label Accessible from Customer Portal | boolean |  | Help: The return label can be downloaded by the customer from the customer portal. |
| `supports_shipping_insurance` | Supports Shipping Insurance | boolean |  | computed by rule `_compute_supports_shipping_insurance` (not stored) |
| `shipping_insurance` | Insurance Percentage | integer |  | default ; Help: Shipping insurance is a service which may reimburse senders whose parcels are lost, stolen, and/or damaged in transit. |
| `price_rule_ids` | Pricing Rules | one to many | `delivery.price.rule` | inverse field `carrier_id` |
| `fixed_price` | Fixed Price | float |  | computed by rule `_compute_fixed_price` and stored; writable through an inverse rule |
| `route_ids` | Routes | many to many | `stock.route` | restricted by domain `[["shipping_selectable", "=", true]]`; association table `stock_route_shipping` |
| `is_mondialrelay` | Is Mondialrelay | boolean |  | computed by rule `_compute_is_mondialrelay` (not stored); searchable through a search rule |
| `mondialrelay_brand` | Brand Code | single line text |  | default `BDTEST` |
| `mondialrelay_packagetype` | Mondialrelay Packagetype | single line text |  | default `24R`; visible only to groups `base.group_system` |
| `website_description` | Description for Online Quotations | multi line text |  | related through path `product_id.description_sale` |
| `l10n_ro_edi_stock_partner_id` | Partner | many to one | `res.partner` |  |
| `gelato_shipping_service_type` | Gelato Shipping Service Type | selection |  | required; default `normal` |
| `warehouse_ids` | Stores | many to many | `stock.warehouse` |  |

## Selection values

### `delivery_type` (Provider)

| Value | Label |
|---|---|
| `base_on_rule` | Based on Rules |
| `fixed` | Fixed Price |
| `gelato` | Gelato |
| `in_store` | Pick up in store |

### `integration_level` (Integration Level)

| Value | Label |
|---|---|
| `rate` | Get Rate |
| `rate_and_ship` | Get Rate and Create Shipment |

### `invoice_policy` (Invoicing Policy)

| Value | Label |
|---|---|
| `estimated` | Estimated cost |
| `real` | Real cost |

### `gelato_shipping_service_type` (Gelato Shipping Service Type)

| Value | Label |
|---|---|
| `normal` | Standard Delivery |
| `express` | Express Delivery |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_margin_not_under_100_percent` | Constraint | `CHECK (margin >= -1)` | Margin cannot be lower than -100% | `delivery` |
| `_shipping_insurance_is_percentage` | Constraint | `CHECK(shipping_insurance >= 0 AND shipping_insurance <= 100)` | The shipping insurance must be a percentage between 0 and 100. | `delivery` |

## Operations (63)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_tags` | validation | self | `delivery` | constrains: `must_have_tag_ids`, `excluded_tag_ids` |  |
| `_compute_weight_uom_name` | computation | self | `delivery` |  |  |
| `_compute_volume_uom_name` | computation | self | `delivery` |  |  |
| `_compute_can_generate_return` | computation | self | `delivery` | depends: `delivery_type` |  |
| `_compute_supports_shipping_insurance` | computation | self | `delivery` | depends: `delivery_type` |  |
| `toggle_prod_environment` | operation | self | `delivery` |  |  |
| `toggle_debug` | operation | self | `delivery` |  |  |
| `install_more_provider` | operation | self | `delivery` |  |  |
| `_is_available_for_order` | internal rule | self, order | `delivery`, `sale_gelato` |  | Override of `delivery` to exclude regular delivery methods from Gelato orders and Gelato delivery methods from non-Gelato orders.  :param sale.order order: The current order. :return: Whether the delivery method is available for the order. :rtype: bool |
| `available_carriers` | operation | self, partner, source | `delivery`, `sale_gelato` |  | Override of `delivery` to filter out regular delivery methods from Gelato orders and Gelato delivery methods from non-Gelato orders.  :param res.partner partner: The partner to check. :param sale.order or stock.picking source: The current order or stock transfer. :return: The available delivery methods. :rtype: delivery.carrier |
| `_match` | internal rule | self, partner, source | `delivery` |  |  |
| `_match_address` | internal rule | self, partner | `delivery` |  |  |
| `_match_must_have_tags` | internal rule | self, source | `delivery` |  |  |
| `_match_excluded_tags` | internal rule | self, source | `delivery` |  |  |
| `_match_weight` | internal rule | self, source | `delivery` |  |  |
| `_match_volume` | internal rule | self, source | `delivery` |  |  |
| `_onchange_integration_level` | on change | self | `delivery` | onchange: `integration_level` |  |
| `_onchange_can_generate_return` | on change | self | `delivery` | onchange: `can_generate_return` |  |
| `_onchange_return_label_on_delivery` | on change | self | `delivery` | onchange: `return_label_on_delivery` |  |
| `_onchange_country_ids` | on change | self | `delivery` | onchange: `country_ids` |  |
| `copy_data` | lifecycle override | self, default | `delivery` |  |  |
| `_get_delivery_type` | preparation rule | self | `delivery` |  | Return the delivery type.  This method needs to be overridden by a delivery carrier module if the delivery type is not stored on the field `delivery_type`. |
| `_apply_margins` | internal rule | self, price, order | `delivery` |  |  |
| `rate_shipment` | operation | self, order | `delivery` |  | Compute the price of the order shipment  :param order: record of sale.order :returns: a dict with structure   ::      {'success': boolean,      'price': a float,      'error_message': a string containing an error message,      'warning_message': a string containing a warning message} :rtype: dict |
| `log_xml` | operation | self, xml_string, func | `delivery` |  |  |
| `_compute_fixed_price` | computation | self | `delivery` | depends: `product_id.list_price`, `product_id.product_tmpl_id.list_price` |  |
| `_set_product_fixed_price` | internal rule | self | `delivery` |  |  |
| `fixed_rate_shipment` | operation | self, order | `delivery` |  |  |
| `base_on_rule_rate_shipment` | operation | self, order | `delivery` |  |  |
| `_get_conversion_currencies` | preparation rule | self, order, conversion | `delivery` |  |  |
| `_compute_currency` | computation | self, order, price, conversion | `delivery` |  |  |
| `_get_price_available` | preparation rule | self, order | `delivery` |  |  |
| `_get_price_dict` | preparation rule | self, total, weight, volume, quantity, wv | `delivery` |  | Hook allowing to retrieve dict to be used in _get_price_from_picking() function. Hook to be overridden when we need to add some field to product and use it in variable factor from price rules. |
| `_get_price_from_picking` | preparation rule | self, total, weight, volume, quantity, wv | `delivery` |  |  |
| `send_shipping` | operation | self, pickings | `stock_delivery` |  | Send the package to the service provider  :param pickings: A recordset of pickings :returns: A list of dictionaries (one per picking) containing of     the form::                   { 'exact_price': price,                    'tracking_number': number } :rtype: list[dict] \| None |
| `get_return_label` | operation | self, pickings, tracking_number, origin_date | `stock_delivery` |  |  |
| `get_return_label_prefix` | operation | self | `stock_delivery` |  |  |
| `_get_delivery_label_prefix` | preparation rule | self | `stock_delivery` |  |  |
| `_get_delivery_doc_prefix` | preparation rule | self | `stock_delivery` |  |  |
| `get_tracking_link` | operation | self, picking | `stock_delivery` |  | Ask the tracking link to the service provider  :param picking: record of stock.picking :returns: an URL containing the tracking link or None :rtype: str \| None |
| `cancel_shipment` | operation | self, pickings | `stock_delivery` |  | Cancel a shipment  :param pickings: A recordset of pickings |
| `_get_default_custom_package_code` | preparation rule | self | `stock_delivery` |  | Some delivery carriers require a prefix to be sent in order to use custom packages (ie not official ones). This optional method will return it as a string. |
| `_get_packages_from_order` | preparation rule | self, order, default_package_type | `stock_delivery` |  |  |
| `_get_packages_from_picking` | preparation rule | self, picking, default_package_type | `stock_delivery` |  |  |
| `_get_commodities_from_order` | preparation rule | self, order | `stock_delivery` |  |  |
| `_get_commodities_from_stock_move_lines` | preparation rule | self, move_lines | `stock_delivery` |  |  |
| `_product_price_to_company_currency` | internal rule | self, quantity, product, company | `stock_delivery` |  |  |
| `fixed_send_shipping` | operation | self, pickings | `stock_delivery` |  |  |
| `fixed_get_tracking_link` | operation | self, picking | `delivery_mondialrelay`, `stock_delivery` |  |  |
| `fixed_cancel_shipment` | operation | self, pickings | `stock_delivery` |  |  |
| `base_on_rule_send_shipping` | operation | self, pickings | `stock_delivery` |  |  |
| `base_on_rule_get_tracking_link` | operation | self, picking | `delivery_mondialrelay`, `stock_delivery` |  |  |
| `base_on_rule_cancel_shipment` | operation | self, pickings | `stock_delivery` |  |  |
| `_compute_is_mondialrelay` | computation | self | `delivery_mondialrelay` | depends: `product_id.default_code` |  |
| `_search_is_mondialrelay` | search rule | self, operator, value | `delivery_mondialrelay` |  |  |
| `gelato_rate_shipment` | operation | self, order | `sale_gelato` |  | Fetch the Gelato delivery price based on products, quantity and address.  This method is called by `delivery`'s `rate_shipment` method.  Note: `self._ensure_one()` from `rate_shipment`  :param sale.order order: The order for which to fetch the delivery price. :return: The shipment rate request results. :rtype: dict |
| `_check_in_store_dm_has_warehouses_when_published` | validation | self | `website_sale_collect` | constrains: `delivery_type`, `is_published`, `warehouse_ids` |  |
| `_check_warehouses_have_same_company` | validation | self | `website_sale_collect` | constrains: `delivery_type`, `company_id`, `warehouse_ids` |  |
| `create` | lifecycle override | self, vals_list | `website_sale_collect` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website_sale_collect` |  |  |
| `_get_in_store_default_vals` | preparation rule |  | `website_sale_collect` |  |  |
| `_in_store_get_close_locations` | internal rule | self, partner_address, product_id | `website_sale_collect` |  | Get the formatted close pickup locations sorted by distance to the partner address.  :param res.partner partner_address: The address to use to sort the pickup locations. :param str product_id: The product whose product page was used to open the location                        selector, if any, as a `product.product` id. :return: The sorted and formatted close pickup locations. :rtype: list[dict] |
| `in_store_rate_shipment` | operation | self, *_args | `website_sale_collect` |  |  |

## Validation and error messages (12)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_tags` | UserError | Carrier %s cannot have the same tag in both Must Have Tags and Excluded Tags. | `delivery` |
| `_match_must_have_tags` | UserError | Invalid source document type | `delivery` |
| `_match_excluded_tags` | UserError | Invalid source document type | `delivery` |
| `_match_weight` | UserError | Invalid source document type | `delivery` |
| `_match_volume` | UserError | Invalid source document type | `delivery` |
| `_get_price_from_picking` | UserError | Not available for current order | `delivery` |
| `_get_packages_from_order` | UserError | The package cannot be created because the total weight of the products in the picking is 0.0 %s | `stock_delivery` |
| `_get_packages_from_picking` | UserError | The package cannot be created because the total weight of the products in the picking is 0.0 %s | `stock_delivery` |
| `base_on_rule_send_shipping` | ValidationError | There is no matching delivery rule. | `stock_delivery` |
| `available_carriers` | UserError | Invalid source document type | `sale_gelato` |
| `_check_in_store_dm_has_warehouses_when_published` | ValidationError | The delivery method must have at least one warehouse to be published. | `website_sale_collect` |
| `_check_warehouses_have_same_company` | ValidationError | The delivery method and a warehouse must share the same company | `website_sale_collect` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `delivery` |
| `base.group_system` | no | yes | no | no | `delivery` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `delivery` |
| `base.group_partner_manager` | no | yes | no | no | `delivery` |
| `stock.group_stock_user` | no | yes | no | no | `stock_delivery` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock_delivery` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Delivery Carrier multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (13)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `delivery.view_delivery_carrier_tree` | list |  | `sequence`, `name`, `delivery_type`, `company_id`, `country_ids`, `max_weight`, `max_volume`, `must_have_tag_ids`, `excluded_tag_ids` |  |  | `delivery` |
| `delivery.view_delivery_carrier_search` | search |  | `name`, `delivery_type` |  | `Archived`, `Provider` | `delivery` |
| `delivery.view_delivery_carrier_form` | form |  | `name`, `active`, `company_id`, `prod_environment`, `debug_logging`, `delivery_type`, `allow_cash_on_delivery`, `integration_level`, `company_id`, `currency_id`, `fixed_price`, `margin`, `fixed_margin`, `free_over`, `amount`, `product_id`, `tracking_url`, `invoice_policy`, `supports_shipping_insurance`, `shipping_insurance`, `price_rule_ids`, `country_ids`, `state_ids`, `zip_prefix_ids`, `max_weight`, `weight_uom_name`, `max_volume`, `volume_uom_name`, `must_have_tag_ids`, `excluded_tag_ids`, `carrier_description` | `toggle_prod_environment`, `toggle_prod_environment`, `toggle_debug`, `toggle_debug`, `Install more Providers` |  | `delivery` |
| `delivery_mondialrelay.view_delivery_carrier_form_provider_mondialrelay` | field | `delivery.view_delivery_carrier_form` | `product_id`, `is_mondialrelay`, `mondialrelay_brand` |  |  | `delivery_mondialrelay` |
| `delivery_mondialrelay.view_delivery_carrier_tree_provider_mondialrelay` | field | `delivery.view_delivery_carrier_tree` | `country_ids` |  |  | `delivery_mondialrelay` |
| `l10n_ro_edi_stock.l10n_ro_edi_stock_view_delivery_carrier_form` | xpath | `delivery.view_delivery_carrier_form` | `l10n_ro_edi_stock_partner_id` |  |  | `l10n_ro_edi_stock` |
| `sale_gelato.delivery_carrier_form` | button | `delivery.view_delivery_carrier_form` |  | `toggle_prod_environment` |  | `sale_gelato` |
| `stock_delivery.view_delivery_carrier_form_inherit_stock_delivery` | xpath | `delivery.view_delivery_carrier_form` | `route_ids` |  |  | `stock_delivery` |
| `website_sale.view_delivery_carrier_form_website_delivery` | field | `delivery.view_delivery_carrier_form` | `company_id`, `website_id` |  |  | `website_sale` |
| `website_sale.view_delivery_carrier_tree` | field | `delivery.view_delivery_carrier_tree` | `delivery_type`, `is_published`, `website_id` |  |  | `website_sale` |
| `website_sale.view_delivery_carrier_search` | filter | `delivery.view_delivery_carrier_search` |  |  | `inactive`, `Published` | `website_sale` |
| `website_sale_collect.delivery_carrier_form` | field | `delivery.view_delivery_carrier_form` | `allow_cash_on_delivery` |  |  | `website_sale_collect` |
| `website_sale_mondialrelay.delivery_carrier_view_search` | xpath | `delivery.view_delivery_carrier_search` |  |  | `Mondial Relay` | `website_sale_mondialrelay` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `delivery.action_delivery_carrier_form` | Delivery Methods | list,form |  | `{'search_default_group_by_provider': True}` |  | `delivery` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `stock_delivery.menu_action_delivery_carrier_form` |  | `stock.menu_delivery` | `delivery.action_delivery_carrier_form` | 1 |  |
| `website_sale.menu_ecommerce_delivery` |  |  | `delivery.action_delivery_carrier_form` | 90 |  |

Machine-readable definition: `../../../schemas/data/entities/delivery.carrier.json`; views: `../../../schemas/interfaces/views/delivery.carrier.json`.
