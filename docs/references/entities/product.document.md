# Product Document (`product.document`)

**Transport name:** `product.document`  
**Storage name:** `product_document`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `sale`, `website_sale`, `mrp`, `sale_gelato`, `sale_pdf_quote_builder`, `website_sale_gelato`

Description: Product Document

## Identity and behavior

- Delegation inheritance: embeds `ir.attachment` through field `ir_attachment_id`
- Default ordering: `sequence, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `ir_attachment_id` | Related attachment | many to one | `ir.attachment` | required; on delete of the target: cascade |
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | default `10` |
| `attached_on_sale` | Sale : Visible at | selection |  | required; default `hidden`; visible only to groups `sales_team.group_sale_salesman`; on delete of the target: {"inside": "set default"}; Help: Allows you to share the document with your customers within a sale. Leave it empty if you don't want to share this document with sales customer. On quote: the document will be sent to and accessible by customers at any time. e.g. this option can be useful to share Product description files. On order confirmation: the document will be sent to and accessible by customers. e.g. this option can be useful to share User Manual or digital content bought on ecommerce.  Inside quote: The document will be included in the pdf of the quotation and sale order between the header pages and the quote table.; extended by packages `sale_pdf_quote_builder` |
| `shown_on_product_page` | Publish on website | boolean |  |  |
| `attached_on_mrp` | manufacturing : Visible at | selection |  | required; default computed dynamically (lambda self: self._default_attached_on_mrp()); Help: Leave hidden if document only accessible on product form. Select Bill of Materials to visualise this document as a product attachment when this product is in a bill of material. |
| `is_gelato` | Is Gelato | boolean |  | read only |
| `form_field_ids` | Form Fields Included | many to many | `sale.pdf.form.field` | computed by rule `_compute_form_field_ids` and stored; restricted by domain `[["document_type", "=", "product_document"]]` |

## Selection values

### `attached_on_sale` (Sale : Visible at)

| Value | Label |
|---|---|
| `hidden` | Hidden |
| `quotation` | On quote |
| `sale_order` | On confirmed order |
| `inside` | Inside quote pdf |

### `attached_on_mrp` (manufacturing : Visible at)

| Value | Label |
|---|---|
| `hidden` | Hidden |
| `bom` | Bill of Materials |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_url` | on change | self | `product` | onchange: `url` |  |
| `create` | lifecycle override | self, vals_list | `product` | model_create_multi |  |
| `copy_data` | lifecycle override | self, default | `product` |  |  |
| `unlink` | lifecycle override | self | `product` |  |  |
| `_unsupported_product_product_document_on_ecommerce` | validation | self | `website_sale` | constrains: `res_model`, `shown_on_product_page` |  |
| `_default_attached_on_mrp` | preparation rule | self | `mrp` |  |  |
| `_gelato_prepare_file_payload` | internal rule | self | `sale_gelato` |  | Create the payload for a single file of an 'orders' request.  :return: The file payload. :rtype: dict |
| `_check_attached_on_and_datas_compatibility` | validation | self | `sale_pdf_quote_builder` | constrains: `attached_on_sale`, `datas`, `type` |  |
| `_compute_form_field_ids` | computation | self | `sale_pdf_quote_builder` | depends: `datas`, `attached_on_sale` |  |
| `action_open_pdf_form_fields` | user action | self | `sale_pdf_quote_builder` |  |  |
| `_check_product_is_unpublished_before_removing_print_images` | validation | self | `website_sale_gelato` | constrains: `datas` |  |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_onchange_url` | ValidationError | Please enter a valid URL. Example: https://www.the system.com  Invalid URL: %s | `product` |
| `_unsupported_product_product_document_on_ecommerce` | ValidationError | Documents shown on product page cannot be restricted to a specific variant | `website_sale` |
| `_gelato_prepare_file_payload` | UserError | Print images must be set on products before they can be ordered. | `sale_gelato` |
| `_check_attached_on_and_datas_compatibility` | ValidationError | When attached inside a quote, the document must be a file, not a URL. | `sale_pdf_quote_builder` |
| `_check_attached_on_and_datas_compatibility` | ValidationError | Only PDF documents can be attached inside a quote. | `sale_pdf_quote_builder` |
| `_check_product_is_unpublished_before_removing_print_images` | ValidationError | Products must be unpublished before print images can be removed. | `website_sale_gelato` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Product multi-company | global (all users) | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |

## Views (15)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.product_document_form` | sheet | `product.product_document_form` | `attached_on_mrp` |  |  | `mrp` |
| `product.product_document_form` | form |  | `res_model`, `name`, `type`, `datas`, `url`, `res_name`, `res_name`, `company_id`, `create_uid`, `create_date` |  |  | `product` |
| `product.product_document_kanban` | kanban |  | `sequence`, `ir_attachment_id`, `mimetype`, `type`, `name`, `active`, `res_model`, `name`, `url`, `res_name` |  |  | `product` |
| `product.product_document_list` | list |  | `res_model`, `sequence`, `name`, `company_id` |  |  | `product` |
| `product.product_document_search` | search |  | `name` |  | `Archived`, `Documents of this variant`, `All` | `product` |
| `sale.product_document_form` | sheet | `product.product_document_form` | `attached_on_sale` |  |  | `sale` |
| `sale.product_document_kanban` | xpath | `product.product_document_kanban` | `attached_on_sale` |  |  | `sale` |
| `sale.product_document_list` | field | `product.product_document_list` | `name`, `attached_on_sale` |  |  | `sale` |
| `sale.product_document_search` | search | `product.product_document_search` |  |  | `Quotation`, `Sales Order` | `sale` |
| `sale_gelato.product_document_form` | form |  | `datas` |  |  | `sale_gelato` |
| `sale_pdf_quote_builder.product_document_form` | field | `product.product_document_form` | `datas` | `Configure dynamic fields` |  | `sale_pdf_quote_builder` |
| `website_sale.product_document_form` | sheet | `sale.product_document_form` | `shown_on_product_page` |  |  | `website_sale` |
| `website_sale.product_document_kanban` | xpath | `sale.product_document_kanban` | `shown_on_product_page` |  |  | `website_sale` |
| `website_sale.product_document_list` | field | `sale.product_document_list` | `attached_on_sale`, `shown_on_product_page` |  |  | `website_sale` |
| `website_sale.product_document_search` | search | `sale.product_document_search` |  |  | `Show on Ecommerce` | `website_sale` |

Machine-readable definition: `../../../schemas/data/entities/product.document.json`; views: `../../../schemas/interfaces/views/product.document.json`.
