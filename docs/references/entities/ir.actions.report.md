# Report Action (`ir.actions.report`)

**Transport name:** `ir.actions.report`  
**Storage name:** `ir_act_report_xml`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `account`, `account_edi`, `account_edi_ubl_cii`, `sale`, `stock`, `hr_expense`, `l10n_din5008`, `l10n_ch`, `l10n_de`, `purchase`, `l10n_th`, `sale_pdf_quote_builder`, `snailmail`

Description: Report Action

## Identity and behavior

- Mixins (classical inheritance): `ir.actions.actions`
- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `type` | Type | single line text |  | default `ir.actions.report` |
| `binding_type` | Binding Type | selection |  | default `report` |
| `model` | Model Name | single line text |  | required |
| `model_id` | Model | many to one | `ir.model` | computed by rule `_compute_model_id` (not stored); searchable through a search rule |
| `report_type` | Report Type | selection |  | required; default `qweb-pdf`; Help: The type of the report that will be rendered, each one having its own rendering method. HTML means the report will be opened directly in your browser PDF means the report will be rendered using Wkhtmltopdf and downloaded by the user. |
| `report_name` | Template Name | single line text |  | required |
| `report_file` | Report File | single line text |  | Help: The path to the main report file (depending on Report Type) or empty if the content is in another field |
| `group_ids` | Groups | many to many | `res.groups` | association table `res_groups_report_rel` |
| `multi` | On Multiple Doc. | boolean |  | Help: If set to true, the action will not be displayed on the right toolbar of a form view. |
| `paperformat_id` | Paper Format | many to one | `report.paperformat` | indexed (btree_not_null) |
| `print_report_name` | Printed Report Name | single line text |  | translatable; Help: This is the filename of the report going to download. Keep empty to not change the report filename. You can use a python expression with the 'object' and 'time' variables. |
| `attachment_use` | Reload from Attachment | boolean |  | Help: If enabled, then the second time the user prints with same attachment name, it returns the previous report. |
| `attachment` | Save as Attachment Prefix | single line text |  | Help: This is the filename of the attachment used to store the printing result. Keep empty to not save the printed reports. You can use a python expression with the object and time variables. |
| `domain` | Filter domain | single line text |  | Help: If set, the action will only appear on records that matches the domain. |
| `is_invoice_report` | Invoice report | boolean |  |  |

## Selection values

### `report_type` (Report Type)

| Value | Label |
|---|---|
| `qweb-html` | HTML |
| `qweb-pdf` | PDF |
| `qweb-text` | Text |

## Operations (46)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_model_id` | computation | self | `base` | depends: `model` |  |
| `_search_model_id` | search rule | self, operator, value | `base` |  |  |
| `_get_readable_fields` | preparation rule | self | `base` |  |  |
| `associated_view` | operation | self | `base` |  | Used in the ir.actions.report form view in order to search naively after the view(s) used in the rendering. |
| `create_action` | operation | self | `base` |  | Create a contextual action for each report. |
| `unlink_action` | operation | self | `base` |  | Remove the contextual actions created for the reports. |
| `retrieve_attachment` | operation | self, record | `base`, `snailmail` |  | Retrieve an attachment for a specific record.  :param record: The record owning of the attachment. :return: A recordset of length <=1 or None |
| `get_wkhtmltopdf_state` | operation | self | `base` | model | Get the current state of wkhtmltopdf: install, ok, upgrade, workers or broken. * install: Starting state. * upgrade: The binary is an older version (< 0.12.0). * ok: A binary was found with a recent version (>= 0.12.0). * workers: Not enough workers found to perform the pdf rendering process (< 2 workers). * broken: A binary was found but not responding.  :return: wkhtmltopdf_state |
| `get_paperformat` | operation | self | `base`, `l10n_ch`, `l10n_din5008`, `snailmail` |  |  |
| `get_paperformat_by_xmlid` | operation | self, xml_id | `base` |  |  |
| `_get_layout` | preparation rule | self | `base` |  |  |
| `_get_report_url` | preparation rule | self, layout | `base` |  |  |
| `_build_wkhtmltopdf_args` | internal rule | self, paperformat_id, landscape, specific_paperformat_args, set_viewport_size | `base` | model | Build arguments understandable by wkhtmltopdf bin.  :param paperformat_id: A report.paperformat record. :param landscape: Force the report orientation to be landscape. :param specific_paperformat_args: A dictionary containing prioritized wkhtmltopdf arguments. :param set_viewport_size: Enable a viewport sized '1024x1280' or '1280x1024' depending of landscape arg. :return: A list of string representing the wkhtmltopdf process command args. |
| `_prepare_html` | preparation rule | self, html, report_model | `base` |  | Divide and recreate the header/footer html by merging all found in html. The bodies are extracted and added to a list. Then, extract the specific_paperformat_args. The idea is to put all headers/footers together. Then, we will use a javascript trick (see minimal_layout template) to set the right header/footer during the processing of wkhtmltopdf. This allows the computation of multiple reports in a single call to wkhtmltopdf. |
| `_run_wkhtmltoimage` | background operation | self, bodies, width, height, image_format | `base` |  | :param str bodies: valid html documents as strings :param int width: width in pixels :param int height: height in pixels :param image_format: format of the image :type image_format: typing.Literal['jpg', 'png'] |
| `_run_wkhtmltopdf` | background operation | self, bodies, report_ref, header, footer, landscape, specific_paperformat_args, set_viewport_size | `base` | model | Execute wkhtmltopdf as a subprocess in order to convert html given in input into a pdf document.  :param Iterable[str] bodies: The html bodies of the report, one per page. :param report_ref: report reference that is needed to get report paperformat. :param str header: The html header of the report containing all headers. :param str footer: The html footer of the report containing all footers. :param landscape: Force the pdf to be rendered under a landscape format. :param specific_paperformat_args: dict of prioritized paperformat arguments. :param set_viewport_size: Enable a viewport sized '102 |
| `_get_report_from_name` | preparation rule | self, report_name | `base` | model | Get the first record of ir.actions.report having the ``report_name`` as value for the field report_name. |
| `_get_report` | preparation rule | self, report_ref | `base` | model | Get the report (with sudo) from a reference  :param report_ref: can be one of      - ir.actions.report id     - ir.actions.report record     - ir.model.data reference to ir.actions.report     - ir.actions.report report_name |
| `barcode` | operation | self, barcode_type, value, **kwargs | `base` | model |  |
| `get_available_barcode_masks` | operation | self | `base`, `l10n_ch` | model | Hook for extension.  This function returns the available QR-code masks, in the form of a list of (code, mask_function) elements, where code is a string identifying the mask uniquely, and mask_function is a function returning a reportlab Drawing object with the result of the mask, and taking as parameters:      - width of the QR-code, in pixels     - height of the QR-code, in pixels     - reportlab Drawing object containing the barcode to apply the mask on |
| `_render_template` | internal rule | self, template, values | `base` |  | Allow to render a QWeb template python-side. This function returns the 'ir.ui.view' render but embellish it with some variables/methods used in reports. :param values: additional methods/variables used in the rendering :returns: html representation of the template :rtype: bytes |
| `_handle_merge_pdfs_error` | internal rule | self, error, error_stream | `base` |  |  |
| `_merge_pdfs` | internal rule | self, streams, handle_error | `base` | model |  |
| `_render_qweb_pdf_prepare_streams` | internal rule | self, report_ref, data, res_ids | `account_edi_ubl_cii`, `account_edi`, `account`, `base`, `hr_expense`, `l10n_ch`, `purchase`, `sale_pdf_quote_builder`, `sale` |  | Override to add and fill headers, footers and product documents to the sale quotation. |
| `_prepare_pdf_report_attachment_vals_list` | preparation rule | self, report, streams | `base` |  | Hook to prepare attachment values needed for attachments creation during the pdf report generation.  :param report: The report (with sudo) from a reference report_ref. :param streams: Dict of streams for each report containing the pdf content and existing attachments. :return: attachment values list needed for attachments creation. |
| `_pre_render_qweb_pdf` | internal rule | self, report_ref, res_ids, data | `account`, `base`, `l10n_th` |  |  |
| `_render_qweb_pdf` | internal rule | self, report_ref, res_ids, data | `base`, `stock` |  |  |
| `_render_qweb_text` | internal rule | self, report_ref, docids, data | `base` | model |  |
| `_render_qweb_html` | internal rule | self, report_ref, docids, data | `base` | model |  |
| `_get_rendering_context_model` | preparation rule | self, report | `base` |  |  |
| `_get_rendering_context` | preparation rule | self, report, docids, data | `account`, `base`, `l10n_de`, `stock` |  |  |
| `_render` | internal rule | self, report_ref, res_ids, data | `base` | model |  |
| `report_action` | operation | self, docids, data, config | `base` |  | Return an action of type ir.actions.report.  :param docids: id/ids/browse record of the records to print (if not used, pass an empty list) :param data: :param bool config: :rtype: bytes |
| `_action_configure_external_report_layout` | internal rule | self, report_action, xml_id | `base` |  |  |
| `get_valid_action_reports` | operation | self, model, record_ids | `base` |  | Return the list of ids of actions for which the domain is satisfied by at least one record in record_ids. :param model: the model of the records to validate :param record_ids: list of ids of records to validate |
| `_prepare_local_attachments` | preparation rule | self, attachments | `base` | model |  |
| `_is_invoice_report` | internal rule | self, report_ref | `account` |  |  |
| `_get_splitted_report` | preparation rule | self, report_ref, content, report_type | `account` |  |  |
| `_unlink_except_master_tags` | internal rule | self | `account` | ondelete |  |
| `_is_sale_order_report` | internal rule | self, report_ref | `sale` |  |  |
| `apply_qr_code_ch_cross_mask` | operation | self, width, height, barcode_drawing | `l10n_ch` | model |  |
| `_is_purchase_order_report` | internal rule | self, report_ref | `purchase` |  |  |
| `_update_mapping_and_add_pages_to_writer` | internal rule | self, writer, document, form_fields_values_mapping, prefix, order, order_line | `sale_pdf_quote_builder` | model | Update the mapping with the field-value of the document, and add the doc to the writer.  Note: document.ensure_one(), order.ensure_one(), order_line and order_line.ensure_one()  :param PdfFileWriter writer: the writer to which pages needs to be added :param recordset document: the document that needs to be added to the writer and get its                            form fields mapped. Either a quotation.document or a                            product.document. :param dict form_fields_values_mapping: the existing prefixed form field names - values that                                         wi |
| `_get_value_from_path` | preparation rule | self, form_field, order, order_line | `sale_pdf_quote_builder` | model | Get the string value by following the path indicated in the record form_field.  :param recordset form_field: sale.pdf.form.field that has a valid path. :param recordset order: sale.order from where the values and timezone need to be taken :param recordset order_line: sale.order.line from where the values need to be taken                              (optional, only for product.document) :return: value that need to be shown in the final pdf. Multiple values are joined by ', ' :rtype: str |
| `_get_custom_value_from_order` | preparation rule | self, document, form_field_name, order, order_line | `sale_pdf_quote_builder` | model | Get the custom value of a form field directly from the order.  :param recordset document: the document that needs to be added to the writer and get its                            form fields mapped. Either a quotation.document or a                            product.document. :param str form_field_name: the name of the form field as present in the PDF. :param recordset order: the sale order from where to take the existing mapping. :param recordset order_line: the sale order line linked to the document (optional) :return: value that need to be shown in the final pdf. :rtype: str |
| `_add_pages_to_writer` | internal rule | self, writer, document, prefix | `sale_pdf_quote_builder` | model | Add a PDF doc to the writer and fill the form text fields present in the pages if needed.  :param PdfFileWriter writer: the writer to which pages needs to be added :param bytes document: the document to add in the final pdf :param str prefix: the prefix needed to update existing form field name, if any, to be able                    to add the correct values in fields with the same name but on different                    documents, either customizable fields or dynamic fields of different sale                    order lines. (optional) :return: None |

## Validation and error messages (11)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_run_wkhtmltoimage` | UserError | wkhtmltoimage 0.12.0^ is required in order to render images from html | `base` |
| `_run_wkhtmltopdf` | UserError | message | `base` |
| `_run_wkhtmltopdf` | UserError | Tried to convert multiple documents in wkhtmltopdf using unpatched QT | `base` |
| `_handle_merge_pdfs_error` | UserError | the system is unable to merge the generated PDFs. | `base` |
| `_merge_pdfs` | UserError | the system is unable to merge the generated PDFs. | `base` |
| `_render_qweb_pdf_prepare_streams` | UserError | Unable to find Wkhtmltopdf on this system. The PDF can not be created. | `base` |
| `_render_qweb_pdf_prepare_streams` | UserError | Report template “%s” has an issue, please contact your administrator.   Cannot separate file to save as attachment because the report's template does not contain the attributes 'data-oe-model' and 'data-oe-id' as part of the div with 'article' classname. | `base` |
| `_render_qweb_pdf_prepare_streams` | UserError | No original purchase document could be found for any of the selected purchase documents. | `account` |
| `_pre_render_qweb_pdf` | UserError | Only invoices could be printed. | `account` |
| `_unlink_except_master_tags` | UserError | You cannot delete this report (%s), it is used by the accounting PDF generation engine. | `account` |
| `_pre_render_qweb_pdf` | UserError | Only invoices could be printed. | `l10n_th` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |
| `base.group_user` | no | yes | no | no | `mail` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.ir_actions_report_form_inherit_account` | field | `base.act_report_xml_view` | `paperformat_id`, `is_invoice_report` |  |  | `account` |
| `base.act_report_xml_view` | form |  | `name`, `report_type`, `paperformat_id`, `model`, `report_name`, `print_report_name`, `group_ids`, `attachment_use`, `attachment` | `create_action`, `unlink_action`, `associated_view` |  | `base` |
| `base.act_report_xml_view_tree` | list |  | `name`, `model`, `type`, `report_name`, `report_type`, `attachment` |  |  | `base` |
| `base.act_report_xml_search_view` | search |  | `name`, `model` |  | `Report Type`, `Report Model` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_action_report` | Reports |  |  |  |  | `base` |
| `base.reports_action` | Reports | list,form |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.report.json`; views: `../../../schemas/interfaces/views/ir.actions.report.json`.
