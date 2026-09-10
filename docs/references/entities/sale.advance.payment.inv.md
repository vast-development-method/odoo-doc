# Sales Advance Payment Invoice (`sale.advance.payment.inv`)

**Transport name:** `sale.advance.payment.inv`  
**Storage name:** `sale_advance_payment_inv`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sale`  
**Extended by packages:** `l10n_in_sale`, `sale_timesheet`

Description: Sales Advance Payment Invoice

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `advance_payment_method` | Create Invoice | selection |  | required; default `delivered`; Help: A standard invoice is issued with all the order lines ready for invoicing,according to their invoicing policy (based on ordered or delivered quantity). |
| `count` | Order Count | integer |  | computed by rule `_compute_count` (not stored) |
| `sale_order_ids` | Sale Order | many to many | `sale.order` | default computed dynamically (lambda self: self.env.context.get('active_ids')) |
| `has_down_payments` | Has down payments | boolean |  | computed by rule `_compute_has_down_payments` (not stored) |
| `deduct_down_payments` | Deduct down payments | boolean |  | default `True` |
| `amount` | Down Payment | float |  | Help: The percentage of amount to be invoiced in advance. |
| `fixed_amount` | Down Payment Amount (Fixed) | monetary |  | Help: The fixed amount to be invoiced in advance. |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` and stored |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored |
| `amount_invoiced` | Already invoiced | monetary |  | computed by rule `_compute_invoice_amounts` (not stored); Help: Only confirmed down payments are considered. |
| `display_draft_invoice_warning` | Display Draft Invoice Warning | boolean |  | computed by rule `_compute_display_draft_invoice_warning` (not stored) |
| `consolidated_billing` | Consolidated Billing | boolean |  | default `True`; Help: Create one invoice for all orders related to same customer, same invoicing address and same delivery address. |
| `date_start_invoice_timesheet` | Start Date | date |  | Help: Only timesheets not yet invoiced (and validated, if applicable) from this period will be invoiced. If the period is not indicated, all timesheets not yet invoiced (and validated, if applicable) will be invoiced without distinction. |
| `date_end_invoice_timesheet` | End Date | date |  | Help: Only timesheets not yet invoiced (and validated, if applicable) from this period will be invoiced. If the period is not indicated, all timesheets not yet invoiced (and validated, if applicable) will be invoiced without distinction. |
| `invoicing_timesheet_enabled` | Invoicing Timesheet Enabled | boolean |  | computed by rule `_compute_invoicing_timesheet_enabled` and stored |

## Selection values

### `advance_payment_method` (Create Invoice)

| Value | Label |
|---|---|
| `delivered` | Regular invoice |
| `percentage` | Down payment (percentage) |
| `fixed` | Down payment (fixed amount) |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_count` | computation | self | `sale` | depends: `sale_order_ids` |  |
| `_compute_has_down_payments` | computation | self | `sale` | depends: `sale_order_ids` |  |
| `_compute_currency_id` | computation | self | `sale` | depends: `sale_order_ids` |  |
| `_compute_company_id` | computation | self | `sale` | depends: `sale_order_ids` |  |
| `_compute_display_draft_invoice_warning` | computation | self | `sale` | depends: `sale_order_ids` |  |
| `_compute_invoice_amounts` | computation | self | `sale` | depends: `sale_order_ids` |  |
| `_onchange_advance_payment_method` | on change | self | `sale` | onchange: `advance_payment_method` |  |
| `_check_amount_is_positive` | validation | self | `sale` |  |  |
| `create_invoices` | operation | self | `sale` |  |  |
| `view_draft_invoices` | operation | self | `sale` |  |  |
| `_create_invoices` | internal rule | self, sale_orders | `sale_timesheet`, `sale` |  | Override method from sale/wizard/sale_make_invoice_advance.py  When the user want to invoice the timesheets to the SO up to a specific period then we need to recompute the qty_to_invoice for each product_id in sale.order.line, before creating the invoice. |
| `_prepare_down_payment_invoice_values` | preparation rule | self, order, so_lines | `sale` |  | Prepare the values to create a down payment invoice.  :param order:       The current sale order. :param so_lines:    The "fake" down payment SO lines created on the sale order. :return:            The values to create a new invoice. |
| `_prepare_down_payment_invoice_line_values` | preparation rule | self, order, so_line, account | `sale` |  | Prepare the invoice line values to be part of a down payment invoice.  :param order:   The current sale order. :param so_line: The "fake" down payment SO line created on the sale order. :param account: The down payment account to use. :return:        The values to create a new invoice line. |
| `_get_down_payment_account` | preparation rule | self, product | `sale` |  | Retrieve the down payment account to use. :param product: A product. :return: An accounting account or None if not found. |
| `_prepare_invoice_values` | preparation rule | self, order, so_line, accounts | `l10n_in_sale` |  |  |
| `_compute_invoicing_timesheet_enabled` | computation | self | `sale_timesheet` | depends: `sale_order_ids` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_amount_is_positive` | UserError | The value of the down payment amount must be positive. | `sale` |
| `_check_amount_is_positive` | UserError | The value of the down payment amount must be positive. | `sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Sales Advance Payment Invoice Rule | global (all users) | `[('create_uid', '=', user.id)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale.view_sale_advance_payment_inv` | form |  | `display_draft_invoice_warning`, `has_down_payments`, `sale_order_ids`, `count`, `consolidated_billing`, `advance_payment_method`, `company_id`, `currency_id`, `fixed_amount`, `amount`, `amount_invoiced` | `Create Draft`, `Cancel` |  | `sale` |
| `sale_timesheet.sale_advance_payment_inv_timesheet_view_form` | form | `sale.view_sale_advance_payment_inv` |  |  |  | `sale_timesheet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale.action_view_sale_advance_payment_inv` | Create invoice(s) | form |  |  | new | `sale` |

Machine-readable definition: `../../../schemas/data/entities/sale.advance.payment.inv.json`; views: `../../../schemas/interfaces/views/sale.advance.payment.inv.json`.
