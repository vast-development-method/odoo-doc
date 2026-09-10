# Quotation Template (`sale.order.template`)

**Transport name:** `sale.order.template`  
**Storage name:** `sale_order_template`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sale_management`  
**Extended by packages:** `sale_pdf_quote_builder`

Description: Quotation Template

## Identity and behavior

- Default ordering: `sequence, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True`; Help: If unchecked, it will allow you to hide the quotation template without removing it. |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `name` | Quotation Template | single line text |  | required |
| `note` | Terms and conditions | rich text |  | translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `mail_template_id` | Confirmation Mail | many to one | `mail.template` | restricted by domain `[["model", "=", "sale.order"]]`; Help: This e-mail template will be sent on confirmation. Leave empty to send nothing. |
| `number_of_days` | Quotation Duration | integer |  | Help: Number of days for the validity date computation of the quotation |
| `require_signature` | Online Signature | boolean |  | computed by rule `_compute_require_signature` and stored; Help: Request a online signature to the customer in order to confirm orders automatically. |
| `require_payment` | Online Payment | boolean |  | computed by rule `_compute_require_payment` and stored; Help: Request an online payment to the customer in order to confirm orders automatically. |
| `prepayment_percent` | Prepayment percentage | float |  | computed by rule `_compute_prepayment_percent` and stored; Help: The percentage of the amount needed to be paid to confirm quotations. |
| `sale_order_template_line_ids` | Lines | one to many | `sale.order.template.line` | inverse field `sale_order_template_id` |
| `journal_id` | Invoicing Journal | many to one | `account.journal` | value is company dependent; restricted by domain `[["type", "=", "sale"]]`; must belong to the same company; Help: If set, SO with this template will invoice in this journal; otherwise the sales journal with the lowest sequence is used. |
| `quotation_document_ids` | Headers and footers | many to many | `quotation.document` | must belong to the same company; association table `header_footer_quotation_template_rel` |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_require_signature` | computation | self | `sale_management` | depends: `company_id` |  |
| `_compute_require_payment` | computation | self | `sale_management` | depends: `company_id` |  |
| `_compute_prepayment_percent` | computation | self | `sale_management` | depends: `company_id`, `require_payment` |  |
| `_onchange_prepayment_percent` | on change | self | `sale_management` | onchange: `prepayment_percent` |  |
| `_check_company_id` | validation | self | `sale_management` | constrains: `company_id`, `sale_order_template_line_ids` |  |
| `_check_prepayment_percent` | validation | self | `sale_management` | constrains: `prepayment_percent` |  |
| `create` | lifecycle override | self, vals_list | `sale_management` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `sale_management` |  |  |
| `_update_product_translations` | internal rule | self | `sale_management` |  |  |
| `_demo_configure_template` | internal rule | self | `sale_management` | model |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_company_id` | ValidationError | Your template cannot contain products from specific companies if it's shared between companies. Please restrict the template access, or remove those products. | `sale_management` |
| `_check_company_id` | ValidationError | Your template belongs to company %(template_company)s but contains products from company (%(product_company)s) that are not accessible to %(template_company)s. Please change the company of your template or remove the products from other companies. | `sale_management` |
| `_check_company_id` | ValidationError | Your template belongs to company %(template_company)s but contains products from other companies (%(product_company)s) that are not accessible to %(template_company)s. Please change the company of your template or remove the products from other companies. | `sale_management` |
| `_check_prepayment_percent` | ValidationError | Prepayment percentage must be a valid percentage. | `sale_management` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_management` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_management` |
| `base.group_system` | no | yes | no | no | `sale_management` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Quotation Template multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale_management.sale_order_template_view_search` | search |  | `name` |  | `Archived`, `Company` | `sale_management` |
| `sale_management.sale_order_template_view_form` | form |  | `name`, `active`, `number_of_days`, `mail_template_id`, `company_id`, `journal_id`, `require_signature`, `require_payment`, `prepayment_percent`, `sale_order_template_line_ids`, `sequence`, `display_type`, `product_id`, `product_uom_qty`, `name`, `name`, `name`, `display_type`, `is_optional`, `sequence`, `product_id`, `name`, `product_uom_qty`, `product_uom_id`, `note` |  |  | `sale_management` |
| `sale_management.sale_order_template_view_tree` | list |  | `sequence`, `name`, `company_id` |  |  | `sale_management` |
| `sale_pdf_quote_builder.sale_order_template_form` | notebook | `sale_management.sale_order_template_view_form` | `quotation_document_ids` |  |  | `sale_pdf_quote_builder` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale_management.sale_order_template_action` | Quotation Templates | list,form |  |  |  | `sale_management` |

Machine-readable definition: `../../../schemas/data/entities/sale.order.template.json`; views: `../../../schemas/interfaces/views/sale.order.template.json`.
