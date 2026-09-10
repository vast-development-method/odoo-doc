# Quotation's Headers & Footers (`quotation.document`)

**Transport name:** `quotation.document`  
**Storage name:** `quotation_document`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sale_pdf_quote_builder`

Description: Quotation's Headers & Footers

## Identity and behavior

- Delegation inheritance: embeds `ir.attachment` through field `ir_attachment_id`
- Default ordering: `document_type desc, sequence, name`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `ir_attachment_id` | Related attachment | many to one | `ir.attachment` | required; on delete of the target: cascade |
| `document_type` | Document Type | selection |  | required; default `header` |
| `active` | Active | boolean |  | default `True`; Help: If unchecked, it will allow you to hide the header or footer without removing it. |
| `sequence` | Sequence | integer |  | default `10` |
| `quotation_template_ids` | Quotation Templates | many to many | `sale.order.template` | visible only to groups `sales_team.group_sale_salesman`; must belong to the same company; association table `header_footer_quotation_template_rel` |
| `form_field_ids` | Form Fields Included | many to many | `sale.pdf.form.field` | computed by rule `_compute_form_field_ids` and stored; restricted by domain `[["document_type", "=", "quotation_document"]]` |
| `add_by_default` | Add By Default | boolean |  | default ; Help: If checked, this header or footer will be added by default on new quotes. |

## Selection values

### `document_type` (Document Type)

| Value | Label |
|---|---|
| `header` | Header |
| `footer` | Footer |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_pdf_validity` | validation | self | `sale_pdf_quote_builder` | constrains: `datas` |  |
| `_compute_form_field_ids` | computation | self | `sale_pdf_quote_builder` | depends: `datas` |  |
| `action_open_pdf_form_fields` | user action | self | `sale_pdf_quote_builder` |  |  |
| `create` | lifecycle override | self, vals_list | `sale_pdf_quote_builder` | model_create_multi |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_pdf_validity` | ValidationError | Only PDF documents can be used as header or footer. | `sale_pdf_quote_builder` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_pdf_quote_builder` |
| `base.group_user` | no | yes | no | no | `sale_pdf_quote_builder` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Quotation document multi-company rule | global (all users) | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale_pdf_quote_builder.quotation_document_form` | form |  | `name`, `document_type`, `datas`, `quotation_template_ids`, `add_by_default`, `company_id`, `form_field_ids`, `create_uid`, `create_date` | `Configure dynamic fields` |  | `sale_pdf_quote_builder` |
| `sale_pdf_quote_builder.quotation_document_kanban` | kanban |  | `sequence`, `ir_attachment_id`, `mimetype`, `document_type`, `name`, `active`, `name`, `document_type`, `quotation_template_ids` |  |  | `sale_pdf_quote_builder` |
| `sale_pdf_quote_builder.quotation_document_list` | list |  | `sequence`, `name`, `document_type`, `quotation_template_ids`, `add_by_default`, `company_id` |  |  | `sale_pdf_quote_builder` |
| `sale_pdf_quote_builder.quotation_document_search_view` | search |  | `name` |  | `All`, `Archived`, `Add By Default`, `Document type`, `Quotation Template` | `sale_pdf_quote_builder` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale_pdf_quote_builder.quotation_document_action` | Headers/Footers | kanban,list,form |  |  |  | `sale_pdf_quote_builder` |

Machine-readable definition: `../../../schemas/data/entities/quotation.document.json`; views: `../../../schemas/interfaces/views/quotation.document.json`.
