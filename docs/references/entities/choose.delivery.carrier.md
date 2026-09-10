# Delivery Carrier Selection Wizard (`choose.delivery.carrier`)

**Transport name:** `choose.delivery.carrier`  
**Storage name:** `choose_delivery_carrier`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `delivery`  
**Extended by packages:** `stock_delivery`, `delivery_mondialrelay`

Description: Delivery Carrier Selection Wizard

## Fields (21)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `order_id` | Order | many to one | `sale.order` | required; on delete of the target: cascade |
| `partner_id` | Partner | many to one | `res.partner` | required; related through path `order_id.partner_id` |
| `carrier_id` | Shipping Method | many to one | `delivery.carrier` | required; restricted by domain `[('id', 'in', available_carrier_ids)]` |
| `delivery_type` | Delivery Type | selection |  | related through path `carrier_id.delivery_type` |
| `delivery_price` | Delivery Price | float |  |  |
| `display_price` | Cost | float |  | read only |
| `currency_id` | Currency | many to one | `res.currency` | related through path `order_id.currency_id` |
| `company_id` | Company | many to one | `res.company` | related through path `order_id.company_id` |
| `available_carrier_ids` | Available Carriers | many to many | `delivery.carrier` | computed by rule `_compute_available_carrier` (not stored) |
| `invoicing_message` | Invoicing Message | multi line text |  | computed by rule `_compute_invoicing_message` (not stored) |
| `delivery_message` | Delivery Message | multi line text |  | read only |
| `total_weight` | Total Order Weight | float |  | related through path `order_id.shipping_weight` |
| `weight_uom_name` | Weight Unit of measure Name | single line text |  | read only; default computed dynamically (_get_default_weight_uom) |
| `shipping_zip` | Shipping Zip | single line text |  | related through path `order_id.partner_shipping_id.zip` |
| `shipping_country_code` | Shipping Country Code | single line text |  | related through path `order_id.partner_shipping_id.country_id.code` |
| `is_mondialrelay` | Is Mondialrelay | boolean |  | computed by rule `_compute_is_mondialrelay` (not stored) |
| `mondialrelay_last_selected` | Last Relay Selected | single line text |  |  |
| `mondialrelay_last_selected_id` | Mondialrelay Last Selected | single line text |  | computed by rule `_compute_mr_last_selected_id` (not stored) |
| `mondialrelay_brand` | Mondialrelay Brand | single line text |  | related through path `carrier_id.mondialrelay_brand` |
| `mondialrelay_colLivMod` | Mondialrelay colLivMod | single line text |  | related through path `carrier_id.mondialrelay_packagetype` |
| `mondialrelay_allowed_countries` | Mondialrelay Allowed Countries | single line text |  | computed by rule `_compute_mr_allowed_countries` (not stored) |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_weight_uom` | preparation rule | self | `delivery` |  |  |
| `_onchange_carrier_id` | on change | self | `delivery` | onchange: `carrier_id`, `total_weight` |  |
| `_onchange_order_id` | on change | self | `delivery` | onchange: `order_id` |  |
| `_compute_invoicing_message` | computation | self | `delivery`, `stock_delivery` | depends: `carrier_id` |  |
| `_compute_available_carrier` | computation | self | `delivery` | depends: `partner_id` |  |
| `_get_delivery_rate` | preparation rule | self | `delivery` |  |  |
| `update_price` | operation | self | `delivery` |  |  |
| `button_confirm` | user action | self | `delivery_mondialrelay`, `delivery` |  |  |
| `_compute_is_mondialrelay` | computation | self | `delivery_mondialrelay` | depends: `carrier_id` |  |
| `_compute_mr_last_selected_id` | computation | self | `delivery_mondialrelay` | depends: `carrier_id`, `order_id.partner_shipping_id` |  |
| `_compute_mr_allowed_countries` | computation | self | `delivery_mondialrelay` | depends: `carrier_id` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `update_price` | UserError | vals.get('error_message') | `delivery` |
| `button_confirm` | ValidationError | Please, choose a Parcel Point | `delivery_mondialrelay` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `delivery` |
| `stock.group_stock_user` | yes | yes | yes | no | `stock_delivery` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `delivery.choose_delivery_carrier_view_form` | form |  | `carrier_id`, `total_weight`, `weight_uom_name`, `delivery_type`, `currency_id`, `order_id`, `delivery_price`, `display_price`, `invoicing_message`, `delivery_message`, `delivery_message` | `update_price`, `Update`, `Add`, `Add`, `Discard` |  | `delivery` |
| `delivery_mondialrelay.choose_delivery_carrier_view_form` | form | `delivery.choose_delivery_carrier_view_form` | `is_mondialrelay`, `mondialrelay_last_selected`, `mondialrelay_last_selected_id`, `mondialrelay_brand`, `mondialrelay_colLivMod`, `mondialrelay_allowed_countries`, `shipping_zip`, `shipping_country_code` |  |  | `delivery_mondialrelay` |
| `stock_delivery.choose_delivery_carrier_view_form` | xpath | `delivery.choose_delivery_carrier_view_form` |  |  |  | `stock_delivery` |

Machine-readable definition: `../../../schemas/data/entities/choose.delivery.carrier.json`; views: `../../../schemas/interfaces/views/choose.delivery.carrier.json`.
