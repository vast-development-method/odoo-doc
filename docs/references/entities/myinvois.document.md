# MyInvois Document (`myinvois.document`)

**Transport name:** `myinvois.document`  
**Storage name:** `myinvois_document`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_my_edi`  
**Extended by packages:** `l10n_my_edi_pos`

Description: MyInvois Document

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `sequence.mixin`
- Default ordering: `myinvois_issuance_date desc, id desc`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_name` and stored; indexed (trigram); not copied on duplication |
| `active` | Active | boolean |  | default `True` |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `currency_id` | Currency | many to one | `res.currency` | required |
| `company_currency_id` | Company Currency | many to one |  | related through path `company_id.currency_id` |
| `myinvois_issuance_date` | Issuance Date | date |  | read only; not copied on duplication |
| `myinvois_file_id` | Myinvois File | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('myinvois_file_id', 'myinvois_file'), depends=['myinvois_file'], copy=False, export_string_translation=False)` (not stored); not copied on duplication |
| `myinvois_file` | MyInvois extensible markup language File | binary |  | read only; not copied on duplication |
| `myinvois_state` | MyInvois State | selection |  | read only; changes are tracked in the message thread; not copied on duplication; Help: State of this document on the MyInvois portal. A document awaiting validation will be automatically updated once the validation status is available. |
| `myinvois_error_document_hash` | Document Hash | single line text |  | read only; not copied on duplication |
| `myinvois_retry_at` | Document Retry At | single line text |  | read only; not copied on duplication |
| `myinvois_exemption_reason` | Tax Exemption Reason | single line text |  | Help: Buyer’s sales tax exemption certificate number, special exemption as per gazette orders, etc. Only applicable if you are using a tax with a type 'Exempt'. |
| `myinvois_custom_form_reference` | Customs Form Reference Number | single line text |  | Help: Reference Number of Customs Form No.1, 9, etc. |
| `myinvois_submission_uid` | Submission UID | single line text |  | read only; not copied on duplication; Help: Unique ID assigned to a batch of documents when sent to MyInvois. |
| `myinvois_external_uuid` | MyInvois identifier | single line text |  | read only; indexed; not copied on duplication; Help: Unique ID assigned to a specific document when sent to MyInvois. |
| `myinvois_validation_time` | Validation Time | date and time |  | read only; not copied on duplication |
| `myinvois_document_long_id` | MyInvois Long identifier | single line text |  | read only; not copied on duplication |
| `Invoices` | Invoices | many to many | `account.move` | must belong to the same company; association table `myinvois_document_invoice_rel` |
| `Orders` | Orders | many to many | `pos.order` | must belong to the same company; association table `myinvois_document_pos_order_rel` |
| `pos_config_id` | PoS Config | many to one | `pos.config` | read only |
| `linked_order_count` | Linked Order Count | integer |  | computed by rule `_compute_linked_order_count` (not stored) |
| `pos_order_date_range` | Date Range | single line text |  | computed by rule `_compute_pos_order_date_range` and stored |

## Selection values

### `myinvois_state` (MyInvois State)

| Value | Label |
|---|---|
| `in_progress` | Validation In Progress |
| `valid` | Valid |
| `rejected` | Rejected |
| `invalid` | Invalid |
| `cancelled` | Cancelled |

## State fields

State machine fields of this entity: `myinvois_state`. Transitions are specified in the domain documents.

## Operations (44)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_name` | computation | self | `l10n_my_edi` | depends: `myinvois_issuance_date` | Compute the name by using the sequence mixin. |
| `_compute_linked_attachment_id` | computation | self, attachment_field, binary_field | `l10n_my_edi` |  | Helper to retrieve Attachment from Binary fields This is needed because fields.Many2one('ir.attachment') makes all attachments available to the user. |
| `_compute_display_name` | computation | self | `l10n_my_edi` | depends: `name` |  |
| `_get_starting_sequence` | preparation rule | self | `l10n_my_edi_pos`, `l10n_my_edi` |  | Defines the default sequence to use by MyInvois Documents. |
| `_get_last_sequence_domain` | preparation rule | self, relaxed | `l10n_my_edi` |  | Returns the SQL WHERE statement to use when fetching the latest record with the same sequence, and its params. |
| `_get_sequence_date_range` | preparation rule | self, reset | `l10n_my_edi` |  | Make sure that the sequence date range follows the company's fiscal year |
| `_unlink_check` | internal rule | self | `l10n_my_edi` | ondelete |  |
| `action_submit_to_myinvois` | user action | self | `l10n_my_edi` |  | Submit all new documents in self to MyInvois. This can also be used on invalid documents to re-submit them after correcting the error. |
| `action_update_submission_status` | user action | self | `l10n_my_edi` |  | Fetches the status of all the documents in self. Note that the endpoint reached to do so will differ based on the amount of documents in the recordset. |
| `action_generate_xml_file` | user action | self | `l10n_my_edi` |  | Generate a xml file for each of the MyInvois documents in self. If the document already as a file, the previous file's name is updated to include an (old) tag to avoid confusion in the attachment list. |
| `action_cancel_submission` | user action | self | `l10n_my_edi` |  | Cancel the document on the platform. |
| `action_show_myinvois_documents` | user action | self | `l10n_my_edi_pos`, `l10n_my_edi` |  | Open the documents in self in the correct view based on the amount of records. |
| `_myinvois_get_proxy_user` | internal rule | self | `l10n_my_edi` |  | Models implementing the mixin should define the logic to get the record's proxy user here. Typically, the one linked to the record's company. :return: The proxy user that should be used to send the record to MyInvois. |
| `_myinvois_log_message` | internal rule | self, message, bodies | `l10n_my_edi` |  | Small helper to use when logging in the chatter to automatically broadcast the message to the invoice.  Supports receiving a simple message string, or a dict of bodies targeted to self. |
| `_myinvois_map_error` | internal rule | self, error | `l10n_my_edi` | model | This helper will take in an error code coming from the proxy, and return a translatable error message. |
| `_is_myinvois_side_error` | internal rule | self, error | `l10n_my_edi` | model | Whether this error genuinely comes from MyInvois's own platform (a validation issue, rate limiting, a technical difficulty, ...), as opposed to anything else (our own pre-submission checks, an unexpected error, ...). |
| `_can_commit` | internal rule |  | `l10n_my_edi` |  | Helper to know if we can commit the current transaction or not.  :returns: True if commit is acceptable, False otherwise. |
| `_get_mail_thread_data_attachments` | preparation rule | self | `l10n_my_edi` |  |  |
| `_get_active_myinvois_document` | preparation rule | self, including_in_progress | `l10n_my_edi` |  | Returns the first document in self that is considered active on the platform. An active document is a document that has been successfully sent, but no cancelled.  There are no flows at the moment where we intend to have more than one active document at a time for a specific record.  :param including_in_progress: if set to true, invoices of state in_progress will be included. |
| `_generate_myinvois_qr_code` | internal rule | self | `l10n_my_edi` |  | Generate the qr code for which can be used to access this document. |
| `_is_refund_document` | internal rule | self | `l10n_my_edi` |  | :return: True if this document is linked to a single refund invoice. |
| `_get_rounded_base_lines` | preparation rule | self | `l10n_my_edi` |  | The base lines used when exporting the document will highly differ based on whether this is or not a consolidated invoice, as well as whether this is for PoS.  :return: The rounded base lines to be used when exporting the document. |
| `_is_consolidated_invoice` | internal rule | self | `l10n_my_edi_pos`, `l10n_my_edi` |  | In a few flows, we need to know if we're dealing with a consolidated invoice in order to set the correct customer for example. This method is here for that; in practice we will be dealing with a consolidated invoice when: - The document is linked to multiple records or; - The document is a refund/credit note of another document linked to multiple records.  :return: True if this invoice is a consolidated invoice or the refund of one. |
| `_is_consolidated_invoice_refund` | internal rule | self | `l10n_my_edi_pos`, `l10n_my_edi` |  | :return: True if this document is a refund specifically for a consolidated invoice. |
| `_split_consolidated_invoice_record_in_lines` | internal rule | self | `l10n_my_edi_pos`, `l10n_my_edi` |  | When dealing with consolidated invoices, all continuous records are grouped in a single line, with a split happening only when the continuity is broken (a document was sent individually,...)  The role of this method is to handle this grouping so that it can be used later when preparing the base lines for export.  :return: a list of recordset containing the related records split into one recordset per line. |
| `_get_record_rounded_base_lines` | preparation rule | self, record | `l10n_my_edi_pos`, `l10n_my_edi` |  | Little helper to return the rounded base line for a record. It is extracted in order to allow extending the logic to support other business models. :param record: The record from which to get the base lines. :return: The rounder base line for the provided record. |
| `_myinvois_submit_documents` | internal rule | self, submissions_content | `l10n_my_edi` |  | Contact our IAP service in order to send the current document's xml to the MyInvois API. Only records in self having a xml_file_content in xml_contents will be sent.  Please mind that the logic will commit for each batch being sent to the platform.  :param submissions_content: A dict of the format {record: {'name': '', 'xml': ''}} :return: a dict of potential errors in the format {record: errors_list} |
| `_myinvois_get_submission_status` | internal rule | self | `l10n_my_edi` |  | Fetches the status of the submissions in self.  :return: A dict of the format: {submission_uid: {'error': '', 'statuses': {record: document_statuses}}} |
| `_myinvois_submission_statuses_update` | internal rule | self, with_commit | `l10n_my_edi` |  | Fetches and update the status of a group of documents.  :param with_commit: If True, we will commit after retrieving the status if we can. |
| `_validate_taxes` | internal rule | self | `l10n_my_edi_pos`, `l10n_my_edi` |  | Makes use of account.edi.xml.ubl_myinvois_my to validate the taxes for the records in self. |
| `_myinvois_generate_xml_file` | internal rule | self | `l10n_my_edi` |  | Generate the xml file representing this record(s) attached to this document. |
| `_submit_to_myinvois` | internal rule | self | `l10n_my_edi` |  | Submit the documents in self to MyInvois. This action will re-generate a new XML file, in order to ensure that we always send an up-to-date version. |
| `_myinvois_check_can_update_status` | internal rule | self | `l10n_my_edi` |  | The document status can only be updated (for rejection, or cancellation) up to 72h after the validation time. After that, any update will be rejected by the platform, as you are expected to issue a debit/credit note.  This helper will raise if the status cannot be updated. |
| `_action_myinvois_update_document` | internal rule | self, new_status | `l10n_my_edi` |  | Returns the action to open the status updated wizard for the mode passed in params.  Valid values for new status are 'cancelled' and 'rejected'. |
| `_myinvois_update_document` | internal rule | self, status, reason | `l10n_my_edi` |  | This method will try to update the status of a document on the platform, and if needed also the status in Odoo.  There is no "Rejected" status on the platform. The document stays as 'valid' until action is taken by the vendor. At that point, the invoice will be cancelled if need be by the call to _myinvois_set_state. |
| `_myinvois_set_state` | internal rule | self, state, message | `l10n_my_edi` |  | Helper to call when the state of one or more documents change. It will handle logging a message if needed, updating the state, cancelling the move when required, and update essential fields that should not be forgotten. |
| `_myinvois_set_validation_fields` | internal rule | self, validation_result | `l10n_my_edi` |  |  |
| `_myinvois_single_status_update` | internal rule | self | `l10n_my_edi` |  | Fetches and update the status of a single document. More efficient than using the submission status endpoint. |
| `_myinvois_statuses_update_cron` | internal rule | self | `l10n_my_edi` | model | This cron is based on the recommended method to fetch the status of the documents according to their doc. MAX_SUBMISSION_UPDATE defines how many submissions to process in a single cron run. |
| `_compute_linked_order_count` | computation | self | `l10n_my_edi_pos` |  |  |
| `_compute_pos_order_date_range` | computation | self | `l10n_my_edi_pos` | depends: `pos_order_ids` |  |
| `action_view_linked_orders` | user action | self | `l10n_my_edi_pos` |  | Return the action used to open the order(s) linked to the selected consolidated invoice. |
| `action_open_consolidate_invoice_wizard` | user action | self | `l10n_my_edi_pos` |  | Open the wizard, and set a default date_from/date_to based on the current date as well as already existing consolidated invoices. |
| `_split_pos_orders_in_lines` | internal rule | self, pos_order_ids | `l10n_my_edi_pos` | model | Separate the orders in self into lines as represented in a consolidated invoice, taking care of splitting when needed.  There is no requirement asking to split per sequence (and thus config), but we still do so to make it easier to submit per PoS if wanted.  :param pos_order_ids: The orders to separate. :return: A dict of pos order per config, for each config having a list of recordset each representing a single line in the xml. |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_check` | UserError | You cannot delete a document that is active on MyInvois. You must cancel it first. | `l10n_my_edi` |
| `action_submit_to_myinvois` | UserError | You cannot send this document to MyInvois because the related invoice(s) %s are in draft or canceled state. | `l10n_my_edi` |
| `action_generate_xml_file` | UserError | Error when generating the documents' files:  - %(errors)s | `l10n_my_edi` |
| `_myinvois_get_proxy_user` | UserError | Please register for the E-Invoicing service in the settings first. | `l10n_my_edi` |
| `_submit_to_myinvois` | UserError | errors[self.id]['plain_text_error'] | `l10n_my_edi` |
| `_myinvois_check_can_update_status` | UserError | It has been more than 72h since the document validation, you can no longer cancel it. Instead, you should issue a debit or credit note. | `l10n_my_edi` |
| `_myinvois_check_can_update_status` | UserError | You can only change the state of a document in the valid or rejected states. | `l10n_my_edi` |
| `_myinvois_single_status_update` | UserError | self._myinvois_map_error(result['error']) | `l10n_my_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_my_edi` |
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_my_edi` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| MyInvois Document | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_my_edi.myinvois_document_form_view` | form |  | `myinvois_state`, `display_name`, `myinvois_issuance_date`, `myinvois_submission_uid`, `myinvois_external_uuid`, `myinvois_validation_time`, `myinvois_exemption_reason`, `myinvois_custom_form_reference` | `Submit To MyInvois`, `Update Submission Status`, `Cancel Submission` |  | `l10n_my_edi` |
| `l10n_my_edi.myinvois_document_list_view` | list |  | `display_name`, `myinvois_issuance_date`, `myinvois_state` | `Submit To MyInvois`, `Fetch Status From MyInvois` |  | `l10n_my_edi` |
| `l10n_my_edi_pos.myinvois_document_pos_form_view` | div | `l10n_my_edi.myinvois_document_form_view` | `linked_order_count` | `action_view_linked_orders` |  | `l10n_my_edi_pos` |
| `l10n_my_edi_pos.myinvois_document_pos_list_view` | header | `l10n_my_edi.myinvois_document_list_view` |  | `Consolidate Orders` |  | `l10n_my_edi_pos` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_my_edi_pos.action_consolidated_invoices` | Consolidated Invoices | list,form | `[('pos_config_id', '!=', False)]` |  |  | `l10n_my_edi_pos` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `l10n_my_edi_pos.menu_consolidated_invoices` | Consolidated Invoice | `point_of_sale.menu_point_of_sale` | `l10n_my_edi_pos.action_consolidated_invoices` | 50 |  |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `l10n_my_edi.action_generate_myinvois_document_file` | Generate Document File | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `l10n_my_edi.ir_cron_myinvois_document_sync` | MyInvois: Document Synchronization | 1 hours | `_myinvois_statuses_update_cron` |  |

Machine-readable definition: `../../../schemas/data/entities/myinvois.document.json`; views: `../../../schemas/interfaces/views/myinvois.document.json`.
