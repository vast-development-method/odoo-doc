# Invoices Statistics (`account.invoice.report`)

**Transport name:** `account.invoice.report`  
**Storage name:** `account_invoice_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `sale`, `l10n_latam_invoice_document`, `l10n_ar`

Description: Invoices Statistics

## Identity and behavior

- Default ordering: `invoice_date desc`
- Display name field: `invoice_date`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (31)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_id` | Move | many to one | `account.move` | read only |
| `journal_id` | Journal | many to one | `account.journal` | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `company_currency_id` | Company Currency | many to one | `res.currency` | read only |
| `partner_id` | Partner | many to one | `res.partner` | read only |
| `commercial_partner_id` | Main Partner | many to one | `res.partner` |  |
| `country_id` | Country | many to one | `res.country` |  |
| `invoice_user_id` | Salesperson | many to one | `res.users` | read only |
| `move_type` | Move Type | selection |  | read only |
| `state` | Invoice Status | selection |  | read only |
| `payment_state` | Payment Status | selection |  | read only |
| `fiscal_position_id` | Fiscal Position | many to one | `account.fiscal.position` | read only |
| `invoice_date` | Invoice Date | date |  | read only |
| `quantity` | Product Quantity | float |  | read only |
| `product_id` | Product | many to one | `product.product` | read only |
| `product_uom_id` | Unit | many to one | `uom.uom` | read only |
| `product_categ_id` | Product Category | many to one | `product.category` | read only |
| `invoice_date_due` | Due Date | date |  | read only |
| `account_id` | Revenue/Expense Account | many to one | `account.account` | read only |
| `price_subtotal_currency` | Untaxed Amount in Currency | float |  | read only |
| `price_subtotal` | Untaxed Amount | float |  | read only |
| `price_total` | Total | float |  | read only |
| `price_total_currency` | Total in Currency | float |  | read only |
| `price_average` | Average Price | float |  | read only; aggregated with avg |
| `price_margin` | Margin | float |  | read only |
| `inventory_value` | Inventory Value | float |  | read only |
| `currency_id` | Currency | many to one | `res.currency` | read only |
| `team_id` | Sales Team | many to one | `crm.team` |  |
| `l10n_latam_document_type_id` | Document Type | many to one | `l10n_latam.document.type` | indexed |
| `l10n_ar_state_id` | Delivery Province | many to one | `res.country.state` | read only |
| `date` | Accounting Date | date |  | read only |

## Selection values

### `move_type` (Move Type)

| Value | Label |
|---|---|
| `out_invoice` | Customer Invoice |
| `in_invoice` | Vendor Bill |
| `out_refund` | Customer Credit Note |
| `in_refund` | Vendor Credit Note |

### `state` (Invoice Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `posted` | Open |
| `cancel` | Cancelled |

## State fields

State machine fields of this entity: `state`, `payment_state`. Transitions are specified in the domain documents.

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_table_query` | internal rule | self | `account` |  |  |
| `_select` | internal rule | self | `account`, `l10n_ar`, `l10n_latam_invoice_document`, `sale` | model |  |
| `_from` | internal rule | self | `account`, `l10n_ar` | model |  |
| `_where` | internal rule | self | `account` | model |  |
| `_read_group_select` | internal rule | self, aggregate_spec, query | `account` |  | This override allows us to correctly calculate the average price of products. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.group_account_manager` | no | yes | no | no | `account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Invoice Analysis multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Personal Invoices Analysis | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|', ('invoice_user_id', '=', user.id), ('invoice_user_id', '=', False)]` | True | True | True | True |
| All Invoices Analysis | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_invoice_report_pivot` | pivot |  | `product_categ_id`, `invoice_date`, `price_subtotal` |  |  | `account` |
| `account.view_account_invoice_report_graph` | graph |  | `product_categ_id`, `price_subtotal` |  |  | `account` |
| `account.account_invoice_report_view_tree` | list |  | `move_id`, `journal_id`, `partner_id`, `country_id`, `invoice_date`, `invoice_date_due`, `invoice_user_id`, `product_categ_id`, `product_id`, `company_id`, `price_average`, `quantity`, `price_subtotal_currency`, `price_subtotal`, `price_total`, `price_total_currency`, `price_margin`, `inventory_value`, `state`, `payment_state`, `move_type` |  |  | `account` |
| `account.view_account_invoice_report_search` | search |  | `invoice_date`, `partner_id`, `invoice_user_id`, `product_id`, `product_categ_id` |  | `My Invoices`, `To Invoice`, `Invoiced`, `Customers`, `Vendors`, `Invoices`, `Credit Notes`, `filter_invoice_date`, `invoice_date_due`, `Salesperson`, `Partner`, `Product Category`, `Status`, `Company`, `Date`, `Date`, `Due Date` | `account` |
| `l10n_ar.view_account_invoice_report_search_inherit` | search | `account.view_account_invoice_report_search` | `l10n_ar_state_id` |  | `With Document`, `Accounting Date: This Year` | `l10n_ar` |
| `l10n_latam_invoice_document.view_account_invoice_report_search` | search | `account.view_account_invoice_report_search` | `l10n_latam_document_type_id` |  |  | `l10n_latam_invoice_document` |
| `sale.view_account_invoice_report_search_inherit` | filter | `account.view_account_invoice_report_search` |  |  | `user`, `Sales Team` | `sale` |
| `sale.account_invoice_report_view_tree` | field | `account.account_invoice_report_view_tree` | `invoice_user_id`, `team_id` |  |  | `sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_account_invoice_report_all_supp` | Bills Analysis | graph,pivot |  | `{'search_default_current':1, 'search_default_supplier': 1, 'group_by':['invoice_date:month']}` |  | `account` |
| `account.action_account_invoice_report_all` | Invoices Analysis | graph,pivot |  | `{'search_default_current':1, 'search_default_customer': 1, 'group_by':['invoice_date:month']}` |  | `account` |
| `l10n_ar.action_iibb_sales_by_state_and_account_pivot` | IIBB - Sales by jurisdiction | pivot |  | `{'search_default_current': 1, 'search_default_customer': 1, 'search_default_with_document': 1, 'search_default_company': 1, 'search_default_groupby_l10n_ar_state_id': 2, 'search_default_groupby_account_id': 3, 'search_default_filter_accounting_date_this_year': 1}` |  | `l10n_ar` |
| `l10n_ar.action_iibb_purchases_by_state_and_account_pivot` | IIBB - Purchases by jurisdiction | pivot |  | `{'search_default_current': 1, 'search_default_supplier': 1, 'search_default_with_document': 1, 'search_default_company': 1, 'search_default_groupby_l10n_ar_state_id': 2, 'search_default_groupby_account_id': 3, 'search_default_filter_accounting_date_this_year': 1}` |  | `l10n_ar` |
| `sale.action_account_invoice_report_salesteam` | Invoices Analysis | graph | `[('state', 'not in', ['draft', 'cancel'])]` | `{'search_default_month':1, 'search_default_team_id': [active_id]}` |  | `sale` |

Machine-readable definition: `../../../schemas/data/entities/account.invoice.report.json`; views: `../../../schemas/interfaces/views/account.invoice.report.json`.
