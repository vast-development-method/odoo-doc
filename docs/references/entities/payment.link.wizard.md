# Generate Payment Link (`payment.link.wizard`)

**Transport name:** `payment.link.wizard`  
**Storage name:** `payment_link_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `payment`  
**Extended by packages:** `account_payment`, `sale`

Description: Generate Payment Link

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_model` | Related Document Model | single line text |  | required |
| `res_id` | Related Document identifier | integer |  | required |
| `amount` | Amount | monetary |  | required; currency taken from `currency_id` |
| `amount_max` | Amount Max | monetary |  | currency taken from `currency_id` |
| `currency_id` | Currency | many to one | `res.currency` |  |
| `partner_id` | Partner | many to one | `res.partner` |  |
| `partner_email` | Partner Email | single line text |  | related through path `partner_id.email` |
| `link` | Payment Link | single line text |  | computed by rule `_compute_link` (not stored) |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` (not stored) |
| `warning_message` | Warning Message | single line text |  | computed by rule `_compute_warning_message` (not stored) |
| `invoice_amount_due` | Amount Due | monetary |  | computed by rule `_compute_invoice_amount_due` (not stored); currency taken from `currency_id` |
| `open_installments` | Open Installments | structured document |  |  |
| `open_installments_preview` | Open Installments Preview | rich text |  | computed by rule `_compute_open_installments_preview` (not stored) |
| `display_open_installments` | Display Open Installments | boolean |  | computed by rule `_compute_display_open_installments` (not stored) |
| `has_eligible_epd` | Has Eligible Epd | boolean |  |  |
| `discount_date` | Discount Date | date |  |  |
| `epd_info` | Early Payment Discount Information | single line text |  | computed by rule `_compute_epd_info` (not stored) |
| `amount_paid` | Already Paid | monetary |  | read only |
| `prepayment_amount` | Prepayment Amount | monetary |  | currency taken from `currency_id` |
| `confirmation_message` | Confirmation Message | single line text |  | computed by rule `_compute_confirmation_message` (not stored) |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `payment` | model |  |
| `_compute_warning_message` | computation | self | `account_payment`, `payment`, `sale` | depends: `amount`, `amount_max`; depends: `res_model`, `res_id` |  |
| `_compute_company_id` | computation | self | `payment` | depends: `res_model`, `res_id` |  |
| `_compute_link` | computation | self | `payment` | depends: `amount`, `currency_id`, `partner_id`, `company_id` |  |
| `_prepare_url` | preparation rule | self, base_url, related_document | `account_payment`, `payment`, `sale` |  | Build the URL of the payment link with the website's base URL and return it. :param str base_url: The website's base URL. :param recordset related_document: The record for which the payment link is generated. :return: The URL of the payment link. :rtype: str |
| `_prepare_query_params` | preparation rule | self, related_document | `account_payment`, `payment`, `sale` |  | Prepare the query string params to append to the payment link URL.  Note: self.ensure_one()  :param recordset related_document: The record for which the payment link is generated. :return: The query params of the payment link. :rtype: dict |
| `_prepare_access_token` | preparation rule | self | `account_payment`, `payment` |  | Override of `payment` to generate the access token only based on the amount. |
| `_prepare_anchor` | preparation rule | self | `account_payment`, `payment` |  | Prepare the anchor to append to the payment link.  Note: self.ensure_one()  :return: The anchor of the payment link. :rtype: str |
| `_compute_invoice_amount_due` | computation | self | `account_payment` | depends: `amount_max` |  |
| `_compute_open_installments_preview` | computation | self | `account_payment` | depends: `open_installments` |  |
| `_compute_epd_info` | computation | self | `account_payment` | depends: `amount` |  |
| `_compute_display_open_installments` | computation | self | `account_payment` | depends: `open_installments` |  |
| `_compute_confirmation_message` | computation | self | `sale` | depends: `amount` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `account_payment` |
| `base.group_user` | no | no | no | no | `payment` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_payment.payment_link_wizard__form_inherit_account_payment` | div | `payment.payment_link_wizard_view_form` | `epd_info` |  |  | `account_payment` |
| `payment.payment_link_wizard_view_form` | form |  | `warning_message`, `res_id`, `res_model`, `partner_id`, `partner_email`, `amount`, `amount_max`, `warning_message`, `currency_id`, `link` | `Close` |  | `payment` |
| `sale.payment_link_wizard_view_form` | field | `payment.payment_link_wizard_view_form` | `amount`, `amount_paid` |  |  | `sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account_payment.action_invoice_order_generate_link` | Generate a Payment Link | form |  |  | new | `account_payment` |
| `sale.action_sale_order_generate_link` | Generate a Payment Link | form |  |  | new | `sale` |

Machine-readable definition: `../../../schemas/data/entities/payment.link.wizard.json`; views: `../../../schemas/interfaces/views/payment.link.wizard.json`.
