# Form fields of inside quotation documents. (`sale.pdf.form.field`)

**Transport name:** `sale.pdf.form.field`  
**Storage name:** `sale_pdf_form_field`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sale_pdf_quote_builder`

Description: Form fields of inside quotation documents.

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Form Field Name | single line text |  | required; read only; Help: The form field name as written in the PDF. |
| `document_type` | Document Type | selection |  | required; read only |
| `path` | Path | single line text |  | Help: The path to follow to dynamically fill the form field.  Leave empty to be able to customized it in the quotation form. |
| `product_document_ids` | Product Documents | many to many | `product.document` |  |
| `quotation_document_ids` | Quotation Documents | many to many | `quotation.document` |  |

## Selection values

### `document_type` (Document Type)

| Value | Label |
|---|---|
| `quotation_document` | Header/Footer |
| `product_document` | Product Document |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_name_per_doc_type` | Constraint | `UNIQUE(name, document_type)` | Form field name must be unique for a given document type. | `sale_pdf_quote_builder` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_form_field_name_follows_pattern` | validation | self | `sale_pdf_quote_builder` | constrains: `name` | Ensure the names only contains alphanumerics, hyphens and underscores.  :return: None :raises: ValidationError if the names aren't alphanumerics, hyphens and underscores. |
| `_check_valid_and_existing_paths` | validation | self | `sale_pdf_quote_builder` | constrains: `path` | Verify that the paths exist and are valid.  :return: None :raises: ValidationError if at least one of the paths isn't valid. |
| `_check_document_type_and_document_linked_compatibility` | validation | self | `sale_pdf_quote_builder` | constrains: `document_type`, `product_document_ids`, `quotation_document_ids` |  |
| `_add_basic_mapped_form_fields` | internal rule | self | `sale_pdf_quote_builder` | model |  |
| `_cron_post_upgrade_assign_missing_form_fields` | background operation | self | `sale_pdf_quote_builder` | model |  |
| `_create_or_update_form_fields_on_pdf_records` | internal rule | self, records, doc_type | `sale_pdf_quote_builder` | model |  |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_form_field_name_follows_pattern` | ValidationError | Invalid form field name %(field_name)s. It should only contain alphanumerics, hyphens or underscores. | `sale_pdf_quote_builder` |
| `_check_form_field_name_follows_pattern` | ValidationError | Invalid form field name %(field_name)s. A form field name in a header or a footer can not start with "sol_id_". | `sale_pdf_quote_builder` |
| `_check_valid_and_existing_paths` | ValidationError | Invalid path %(path)s. It should only contain alphanumerics, hyphens, underscores or points. | `sale_pdf_quote_builder` |
| `_check_valid_and_existing_paths` | ValidationError | Please use only relational fields until the last value of your path. | `sale_pdf_quote_builder` |
| `_check_valid_and_existing_paths` | ValidationError | The field %(field_name)s doesn't exist on model %(model_name)s | `sale_pdf_quote_builder` |
| `_check_document_type_and_document_linked_compatibility` | ValidationError | A form field set as used in product documents can't be linked to a quotation document. | `sale_pdf_quote_builder` |
| `_check_document_type_and_document_linked_compatibility` | ValidationError | A form field set as used in quotation documents can't be linked to a product document. | `sale_pdf_quote_builder` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `sale_pdf_quote_builder` |
| `base.group_user` | no | yes | no | no | `sale_pdf_quote_builder` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale_pdf_quote_builder.sale_pdf_form_field_list` | list |  | `name`, `path`, `document_type`, `product_document_ids`, `quotation_document_ids` |  |  | `sale_pdf_quote_builder` |
| `sale_pdf_quote_builder.quotation_document_search` | search |  | `name` |  | `Customizable`, `This document` | `sale_pdf_quote_builder` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `sale_pdf_quote_builder.cron_post_upgrade_assign_missing_form_fields` | Sale Pdf Quote Builder: assign form fields to documents post upgrade | 9999 months | `_cron_post_upgrade_assign_missing_form_fields` |  |

Machine-readable definition: `../../../schemas/data/entities/sale.pdf.form.field.json`; views: `../../../schemas/interfaces/views/sale.pdf.form.field.json`.
