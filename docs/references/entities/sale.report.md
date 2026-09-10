# Sales Analysis Report (`sale.report`)

**Transport name:** `sale.report`  
**Storage name:** `sale_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sale`  
**Extended by packages:** `sale_stock`, `website_sale`, `pos_sale`, `sale_margin`, `pos_sale_margin`, `sale_project`

Description: Sales Analysis Report

## Identity and behavior

- Default ordering: `date desc`
- Display name field: `date`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (45)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Order Reference | single line text |  | read only |
| `date` | Order Date | date and time |  | read only |
| `partner_id` | Customer | many to one | `res.partner` | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `pricelist_id` | Pricelist | many to one | `product.pricelist` | read only |
| `team_id` | Sales Team | many to one | `crm.team` | read only |
| `user_id` | Salesperson | many to one | `res.users` | read only |
| `state` | Status | selection |  | read only; extended by packages `pos_sale` |
| `invoice_status` | Order Invoice Status | selection |  | read only |
| `campaign_id` | Campaign | many to one | `utm.campaign` | read only |
| `medium_id` | Medium | many to one | `utm.medium` | read only |
| `source_id` | Source | many to one | `utm.source` | read only |
| `commercial_partner_id` | Customer Entity | many to one | `res.partner` | read only |
| `country_id` | Customer Country | many to one | `res.country` | read only |
| `industry_id` | Customer Industry | many to one | `res.partner.industry` | read only |
| `partner_zip` | Customer ZIP | single line text |  | read only |
| `state_id` | Customer State | many to one | `res.country.state` | read only |
| `order_reference` | Order | reference |  | aggregated with count_distinct; extended by packages `pos_sale` |
| `categ_id` | Product Category | many to one | `product.category` | read only |
| `product_id` | Product Variant | many to one | `product.product` | read only |
| `product_tmpl_id` | Product | many to one | `product.template` | read only |
| `product_uom_id` | Unit | many to one | `uom.uom` | read only |
| `product_uom_qty` | Qty Ordered | float |  | read only |
| `qty_to_deliver` | Qty To Deliver | float |  | read only |
| `qty_delivered` | Qty Delivered | float |  | read only |
| `qty_to_invoice` | Qty To Invoice | float |  | read only |
| `qty_invoiced` | Qty Invoiced | float |  | read only |
| `price_subtotal` | Untaxed Total | monetary |  | read only |
| `price_total` | Total | monetary |  | read only |
| `untaxed_amount_to_invoice` | Untaxed Amount To Invoice | monetary |  | read only |
| `untaxed_amount_invoiced` | Untaxed Amount Invoiced | monetary |  | read only |
| `line_invoice_status` | Invoice Status | selection |  | read only |
| `weight` | Gross Weight | float |  | read only |
| `volume` | Volume | float |  | read only |
| `price_unit` | Unit Price | float |  | read only; aggregated with avg |
| `discount` | Discount % | float |  | read only; aggregated with avg |
| `discount_amount` | Discount Amount | monetary |  | read only |
| `nbr` | # of Lines | integer |  | read only |
| `currency_id` | Currency | many to one | `res.currency` | read only |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | read only |
| `website_id` | Website | many to one | `website` | read only |
| `is_abandoned_cart` | Abandoned Cart | boolean |  | read only |
| `public_categ_ids` | eCommerce Categories | many to many |  | related through path `product_tmpl_id.public_categ_ids` |
| `margin` | Margin | float |  |  |
| `project_id` | Project | many to one | `project.project` | read only |

## Selection values

### `invoice_status` (Order Invoice Status)

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

### `order_reference` (Order)

| Value | Label |
|---|---|
| `sale.order` | Sales Order |
| `pos.order` | POS Order |

### `line_invoice_status` (Invoice Status)

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

## State fields

State machine fields of this entity: `state`, `invoice_status`, `line_invoice_status`. Transitions are specified in the domain documents.

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_done_states` | preparation rule | self | `pos_sale`, `sale` | model |  |
| `_with_sale` | internal rule | self | `sale` |  |  |
| `_select_sale` | internal rule | self | `sale` |  |  |
| `_case_value_or_one` | internal rule | self, value | `sale` |  |  |
| `_select_additional_fields` | internal rule | self | `sale_margin`, `sale_project`, `sale_stock`, `sale`, `website_sale` |  | Hook to return additional fields SQL specification for select part of the table query.  :returns: mapping field -> SQL computation of field, will be converted to '_ AS _field' in the final table definition :rtype: dict |
| `_from_sale` | internal rule | self | `sale`, `website_sale` |  |  |
| `_where_sale` | internal rule | self | `sale` |  |  |
| `_group_by_sale` | internal rule | self | `sale_stock`, `sale`, `website_sale` |  |  |
| `_query` | internal rule | self | `pos_sale`, `sale` |  |  |
| `_table_query` | internal rule | self | `sale` |  |  |
| `action_open_order` | user action | self | `sale` | readonly |  |
| `_select_pos` | internal rule | self | `pos_sale` |  |  |
| `_available_additional_pos_fields` | internal rule | self | `pos_sale` |  | Hook to replace the additional fields from sale with the one from pos_sale. |
| `_fill_pos_fields` | internal rule | self, additional_fields | `pos_sale_margin`, `pos_sale` |  | Hook to fill additional fields for the pos_sale.  :param additional_fields: Dictionary mapping fields with their values :type additional_fields: dict[str, Any] |
| `_from_pos` | internal rule | self | `pos_sale` |  |  |
| `_where_pos` | internal rule | self | `pos_sale` |  |  |
| `_group_by_pos` | internal rule | self | `pos_sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Sales Order Analysis multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Personal Orders Analysis | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | True | True | True | True |
| All Orders Analysis | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | True | True | True | True |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale.view_order_product_pivot` | pivot |  | `team_id`, `date`, `product_uom_qty` |  |  | `sale` |
| `sale.view_order_product_graph` | graph |  | `date`, `product_uom_qty` |  |  | `sale` |
| `sale.sale_report_graph_pie` | graph | `view_order_product_graph` |  |  |  | `sale` |
| `sale.sale_report_graph_bar` | graph | `view_order_product_graph` |  |  |  | `sale` |
| `sale.sale_report_view_tree` | list |  | `date`, `order_reference`, `product_id`, `partner_id`, `user_id`, `team_id`, `company_id`, `product_uom_qty`, `price_subtotal`, `price_unit`, `price_total`, `state`, `pricelist_id`, `line_invoice_status`, `currency_id` |  |  | `sale` |
| `sale.view_order_product_search` | search |  | `date`, `user_id`, `team_id`, `product_id`, `product_tmpl_id`, `categ_id`, `partner_id`, `country_id`, `industry_id`, `categ_id`, `company_id` |  | `Date`, `Quotations`, `Sales Orders`, `filter_date`, `Order Date: Last 365 Days`, `To Invoice`, `Fully Invoiced`, `Salesperson`, `Sales Team`, `Customer`, `Customer Country`, `Customer Industry`, `Product`, `Product Variant`, `Product Category`, `Status`, `Company`, `Order Date`, `Order Date` | `sale` |
| `website_sale.sale_report_view_search_website` | search |  | `website_id`, `product_id`, `categ_id`, `partner_id`, `country_id`, `company_id` |  | `Confirmed Orders`, `filter_date`, `Last Week`, `Last Month`, `Last Year`, `Website`, `Product`, `Product Category`, `Customer`, `Customer Country`, `Status`, `eCommerce Category`, `Order Date` | `website_sale` |
| `website_sale.sale_report_view_pivot_website` | pivot |  | `date`, `state`, `price_subtotal` |  |  | `website_sale` |
| `website_sale.sale_report_view_graph_website` | graph |  | `date`, `price_subtotal` |  |  | `website_sale` |
| `website_sale.sale_report_view_tree` | field | `sale.sale_report_view_tree` | `order_reference`, `website_id`, `public_categ_ids` |  |  | `website_sale` |
| `website_sale_slides.sale_report_view_graph_slides` | graph |  | `date`, `price_total` |  |  | `website_sale_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale.action_order_report_all` | Sales Analysis | graph,pivot,list,form | `[('state', '!=', 'cancel')]` | `{'search_default_Sales':1,'group_by':[], 'search_default_filter_order_date': 1}` |  | `sale` |
| `sale.action_order_report_salesperson` | Sales Analysis By Salespersons | graph,pivot |  | `{'search_default_User': 1, 'group_by': 'user_id', 'search_default_filter_order_date': 1}` |  | `sale` |
| `sale.action_order_report_products` | Sales Analysis By Products | graph,pivot |  | `{'search_default_Sales': 1, 'search_default_Product': 1, 'group_by': 'product_id', 'search_default_filter_order_date': 1}` |  | `sale` |
| `sale.action_order_report_customers` | Sales Analysis By Customers | graph,pivot |  | `{'search_default_Customer': 1, 'group_by': 'partner_id', 'search_default_filter_order_date': 1}` |  | `sale` |
| `sale.report_all_channels_sales_action` | Sales Analysis | list,pivot,graph,form |  |  |  | `sale` |
| `sale.action_order_report_quotation_salesteam` | Quotations Analysis | graph,list | `[('state','=','draft'),('team_id', '=', active_id)]` | `{'search_default_order_month':1}` |  | `sale` |
| `sale.action_order_report_so_salesteam` | Sales Analysis | graph,list | `[('state','not in',('draft','cancel'))]` | `{             'search_default_Sales': 1,             'search_default_filter_date': 1,             'search_default_team_id': [active_id]}` |  | `sale` |
| `website_sale.sale_report_action_dashboard` | Online Sales Analysis | pivot,graph | `[('website_id', '!=', False)]` | `{'search_default_confirmed': 1}` |  | `website_sale` |
| `website_sale.sale_report_action_carts` | Sales | pivot,graph | `[('website_id', '!=', False)]` |  |  | `website_sale` |
| `website_sale_slides.sale_report_action_slides` | eLearning Revenues | graph,pivot | `[("product_id.channel_ids", "!=", False)]` | `{'group_by': ['date', 'product_id'], 'pivot_measures': ['price_total']}` |  | `website_sale_slides` |

Machine-readable definition: `../../../schemas/data/entities/sale.report.json`; views: `../../../schemas/interfaces/views/sale.report.json`.
