# Attachment (`ir.attachment`)

**Transport name:** `ir.attachment`  
**Storage name:** `ir_attachment`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `bus`, `html_editor`, `mail`, `mail`, `product`, `account`, `account_edi`, `api_doc`, `attachment_indexation`, `cloud_storage`, `cloud_storage_azure`, `cloud_storage_google`, `cloud_storage_migration`, `hr_expense`, `hr_fleet`, `hr_recruitment`, `website`, `website`, `l10n_fr_pdp`, `l10n_in_edi`, `l10n_in_ewaybill`, `l10n_jo_edi`, `l10n_sa`, `l10n_sa_edi`, `mrp`, `website_forum`

Description: Attachment

## Identity and behavior

- Mixins (classical inheritance): `bus.listener.mixin`
- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (30)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `description` | Description | multi line text |  |  |
| `res_name` | Resource Name | single line text |  | computed by rule `_compute_res_name` (not stored) |
| `res_model` | Resource Model | single line text |  |  |
| `res_field` | Resource Field | single line text |  |  |
| `res_id` | Resource identifier | many to one by reference |  |  |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `type` | Type | selection |  | required; default `binary`; on delete of the target: {"cloud_storage": "set url"}; Help: You can either upload a file from your computer or copy/paste an internet link to your file.; extended by packages `cloud_storage` |
| `url` | Url | single line text |  | indexed (btree_not_null); maximum length 1024 |
| `public` | Is public document | boolean |  |  |
| `access_token` | Access Token | single line text |  | visible only to groups `base.group_user` |
| `raw` | File Content (raw) | binary |  | computed by rule `_compute_raw` (not stored); writable through an inverse rule |
| `datas` | File Content (base64) | binary |  | computed by rule `_compute_datas` (not stored); writable through an inverse rule |
| `db_datas` | Database Data | binary |  |  |
| `store_fname` | Stored Filename | single line text |  | indexed |
| `file_size` | File Size | integer |  | read only |
| `checksum` | Checksum/SHA1 | single line text |  | read only; maximum length 40 |
| `mimetype` | Mime Type | single line text |  | read only |
| `index_content` | Indexed Content | multi line text |  | read only |
| `local_url` | Attachment uniform resource locator | single line text |  | computed by rule `_compute_local_url` (not stored) |
| `image_src` | Image Src | single line text |  | computed by rule `_compute_image_src` (not stored) |
| `image_width` | Image Width | integer |  | computed by rule `_compute_image_size` (not stored) |
| `image_height` | Image Height | integer |  | computed by rule `_compute_image_size` (not stored) |
| `original_id` | Original (unoptimized, unresized) attachment | many to one | `ir.attachment` | indexed (btree_not_null) |
| `voice_ids` | Voice | one to many | `discuss.voice.metadata` | inverse field `attachment_id` |
| `thumbnail` | Thumbnail | image |  |  |
| `has_thumbnail` | Has Thumbnail | boolean |  | computed by rule `_compute_has_thumbnail` (not stored) |
| `key` | Key | single line text |  | not copied on duplication; extended by packages `website` |
| `website_id` | Website | many to one | `website` |  |
| `theme_template_id` | Theme Template | many to one | `theme.ir.attachment` | indexed (btree_not_null); not copied on duplication |

## Selection values

### `type` (Type)

| Value | Label |
|---|---|
| `url` | URL |
| `binary` | File |
| `cloud_storage` | Cloud Storage |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_res_idx` | Index | `(res_model, res_id)` |  | `base` |

## Operations (101)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_res_name` | computation | self | `base` |  |  |
| `_storage` | internal rule | self | `base` | model |  |
| `_filestore` | internal rule | self | `base` | model |  |
| `_get_storage_domain` | preparation rule | self | `base` | model |  |
| `force_storage` | operation | self | `base` | model | Force all attachments to be stored in the currently configured storage |
| `_migrate` | internal rule | self | `base` |  |  |
| `_full_path` | internal rule | self, path | `base` | model |  |
| `_get_path` | preparation rule | self, bin_data, sha | `base` | model |  |
| `_file_read` | internal rule | self, fname, size | `base` | model |  |
| `_file_write` | internal rule | self, bin_value, checksum | `base` | model |  |
| `_file_delete` | internal rule | self, fname | `base` | model |  |
| `_mark_for_gc` | internal rule | self, fname | `base` |  | Add ``fname`` in a checklist for the filestore garbage collection. |
| `_gc_file_store` | background operation | self | `base` | autovacuum | Perform the garbage collection of the filestore. |
| `_gc_file_store_unsafe` | background operation | self | `base` |  |  |
| `_compute_datas` | computation | self | `base` | depends: `store_fname`, `db_datas`, `file_size`; depends_context: `bin_size` |  |
| `_compute_raw` | computation | self | `base` | depends: `store_fname`, `db_datas` |  |
| `_get_pdf_raw` | preparation rule | self | `base` |  |  |
| `_inverse_raw` | inverse computation | self | `base` |  |  |
| `_inverse_datas` | inverse computation | self | `base` |  |  |
| `_set_attachment_data` | internal rule | self, asbytes | `base` |  |  |
| `_get_datas_related_values` | preparation rule | self, data, mimetype | `base` |  |  |
| `_compute_checksum` | computation | self, bin_data | `base` |  | compute the checksum for the given datas :param bin_data : datas in its binary form |
| `_same_content` | internal rule | self, bin_data, filepath | `base` | model |  |
| `_compute_mimetype` | computation | self, values | `base` |  | compute the mimetype of the given values :param values : dict of values to create or write an ir_attachment :return mime : string indicating the mimetype, or application/octet-stream by default |
| `_postprocess_contents` | internal rule | self, values | `base` |  |  |
| `_check_contents` | validation | self, values | `base` |  |  |
| `_index` | internal rule | self, bin_data, file_type, checksum | `attachment_indexation`, `base` | model | compute the index content of the given binary data. This is a python implementation of the unix command 'strings'. |
| `get_serving_groups` | operation | self | `base`, `website` | model | An ir.attachment record may be used as a fallback in the http dispatch if its type field is set to "binary" and its url field is set as the request's url. Only the groups returned by this method are allowed to create and write on such records. |
| `_check_serving_attachments` | validation | self | `base` |  |  |
| `_check_circular_attachment` | validation | self | `base` | constrains: `res_model`, `res_id` |  |
| `check` | operation | self, mode, values | `base` | model | Restricts the access to an ir.attachment, according to referred mode |
| `_check_access` | validation | self, operation | `base` |  | Check access for attachments.  Rules: - `public` is always accessible for reading. - If we have `res_model and res_id`, the attachment is accessible if the   referenced model is accessible. Also, when `res_field != False` and   the user is not an administrator, we check the access on the field. - If we don't have a referenced record, the attachment is accessible to   the administrator and the creator of the attachment. |
| `_inaccessible_comodel_records` | internal rule | self, model_and_ids, operation | `base` |  |  |
| `_search` | search rule | self, domain, offset, limit, order, active_test, bypass_access | `base` | model |  |
| `write` | lifecycle override | self, vals | `account`, `base` |  |  |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `unlink` | lifecycle override | self | `account`, `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base`, `hr_expense`, `product`, `website` | model_create_multi | Create product.document for attachments added in products chatters |
| `_post_add_create` | internal rule | self, **kwargs | `account`, `base`, `cloud_storage`, `mail`, `mrp` |  | Overrides behaviour when the attachment is created through the controller |
| `generate_access_token` | operation | self | `base` |  |  |
| `_get_raw_access_token` | preparation rule | self | `base` |  | Return a scoped access token for the `raw` field. The token can be used with `ir_binary._find_record` to bypass access rights.  :rtype: str |
| `create_unique` | operation | self, values_list | `base` | model |  |
| `_generate_access_token` | internal rule | self | `base` |  |  |
| `action_get` | user action | self | `base` | model |  |
| `_get_serve_attachment` | preparation rule | self, url, extra_domain, order | `base`, `website` | model |  |
| `regenerate_assets_bundles` | operation | self | `base` | model |  |
| `_from_request_file` | internal rule | self, file, mimetype, **vals | `base` |  | Create an attachment out of a request file  :param file: the request file :param str mimetype:     * "TRUST" to use the mimetype and file extension from the       request file with no verification.     * "GUESS" to determine the mimetype and file extension on       the file's content. The determined extension is added at       the end of the filename unless the filename already had a       valid extension.     * a mimetype in format "{type}/{subtype}" to force the       mimetype to the given value, it adds the corresponding       file extension at the end of the filename unless the       filen |
| `_to_http_stream` | internal rule | self | `base`, `cloud_storage` |  | Create a :class:`~Stream`: from an ir.attachment record. |
| `_is_remote_source` | internal rule | self | `base` |  |  |
| `_can_return_content` | internal rule | self, field_name, access_token | `base` |  |  |
| `_migrate_remote_to_local` | internal rule | self | `base`, `cloud_storage` |  |  |
| `_bus_channel` | internal rule | self | `bus`, `mail` |  |  |
| `_compute_local_url` | computation | self | `html_editor` |  |  |
| `_compute_image_src` | computation | self | `html_editor` | depends: `mimetype`, `url`, `name` |  |
| `_compute_image_size` | computation | self | `html_editor` | depends: `datas` |  |
| `_get_media_info` | preparation rule | self | `html_editor` |  | Return a dict with the values that we need on the media dialog. |
| `_can_bypass_rights_on_media_dialog` | internal rule | self, **attachment_data | `html_editor`, `website_forum` |  | This method is meant to be overridden, for instance to allow to create image attachment despite the user not allowed to create attachment, eg: - Portal user uploading an image on the forum (bypass acl) - Non admin user uploading an unsplash image (bypass binary/url check) |
| `_to_store_defaults` | internal rule | self, target | `mail` |  |  |
| `_set_voice_metadata` | internal rule | self | `mail` |  |  |
| `_compute_has_thumbnail` | computation | self | `mail` | depends: `thumbnail` |  |
| `_has_attachments_ownership` | internal rule | self, attachment_tokens | `mail` |  | Checks if the current user has ownership of all attachments in the recordset. Ownership is defined as either: - Having 'write' access to the attachment. - Providing a valid, scoped 'attachment_ownership' access token.  :param list attachment_tokens: A list of access tokens |
| `register_as_main_attachment` | operation | self, force | `mail` |  | Registers this attachment as the main one of the model it is attached to.  :param bool force: if set, the method always updates the existing main attachment     otherwise it only sets the main attachment if there is none. |
| `_delete_and_notify` | internal rule | self, message | `mail` |  |  |
| `_get_store_ownership_fields` | preparation rule | self | `mail` |  |  |
| `_get_ownership_token` | preparation rule | self | `mail` |  | Returns a scoped limited access token that indicates ownership of the attachment when using _has_attachments_ownership. If verified by verify_limited_field_access_token, accessing the attachment bypasses the ACLs.  :rtype: str |
| `_get_thumbnail_token` | preparation rule | self | `mail` |  |  |
| `_build_zip_from_attachments` | internal rule | self | `account` |  | Return the zip bytes content resulting from compressing the attachments in `self` |
| `_except_audit_trail` | internal rule | self | `account` | ondelete |  |
| `_unlink_except_government_document` | internal rule | self | `account_edi` | ondelete |  |
| `_gc_doc_index` | background operation | self | `api_doc` | autovacuum | Garbage collect the outdated /doc/index.json attachments. |
| `_index_docx` | internal rule | self, bin_data | `attachment_indexation` |  | Index Microsoft .docx documents |
| `_index_pptx` | internal rule | self, bin_data | `attachment_indexation` |  | Index Microsoft .pptx documents |
| `_index_xlsx` | internal rule | self, bin_data | `attachment_indexation` |  | Index Microsoft .xlsx documents |
| `_index_opendoc` | internal rule | self, bin_data | `attachment_indexation` |  | Index OpenDocument documents (.odt, .ods...) |
| `_index_pdf` | internal rule | self, bin_data | `attachment_indexation` |  | Index PDF documents |
| `copy` | lifecycle override | self, default | `attachment_indexation` |  |  |
| `_generate_cloud_storage_blob_name` | internal rule | self | `cloud_storage` |  | Generate a unique blob name for the attachment  :return: A unique blob name str |
| `_generate_cloud_storage_url` | internal rule | self | `cloud_storage_azure`, `cloud_storage_google`, `cloud_storage` |  | Generate a cloud blob url without signature or token for the attachment. This url is only used to identify the cloud blob.  :return: A cloud blob url str |
| `_get_cloud_storage_download_url_time_to_expiry` | preparation rule | self | `cloud_storage` |  |  |
| `_generate_cloud_storage_download_info` | internal rule | self | `cloud_storage_azure`, `cloud_storage_google`, `cloud_storage` |  | Generate the download info for the public client to directly download the attachment's blob from the cloud storage.  :return: An download_info dictionary containing:      download_url         cloud storage url with permission to download the file     time_to_expiry         the time in seconds before the download url expires |
| `_generate_cloud_storage_upload_info` | internal rule | self | `cloud_storage_azure`, `cloud_storage_google`, `cloud_storage` |  | Generate the upload info for the public client to directly upload a file to the cloud storage.  :return: An upload_info dictionary containing:      upload_url         cloud storage url with permission to upload the file     method         the request method used to upload the file     response_status         the status of the response for a successful upload request     [Optionally] headers         a dictionary of headers to be added to the upload request |
| `_get_cloud_storage_unsupported_models` | preparation rule | self | `cloud_storage` |  |  |
| `_get_cloud_storage_azure_info` | preparation rule | self | `cloud_storage_azure` |  |  |
| `_generate_cloud_storage_azure_url` | internal rule | self, blob_name | `cloud_storage_azure` |  |  |
| `_generate_cloud_storage_azure_sas_url` | internal rule | self, **kwargs | `cloud_storage_azure` |  |  |
| `_get_cloud_storage_google_info` | preparation rule | self | `cloud_storage_google` |  |  |
| `_generate_cloud_storage_google_url` | internal rule | self, blob_name | `cloud_storage_google` |  |  |
| `_generate_cloud_storage_google_signed_url` | internal rule | self, bucket_name, blob_name, **kwargs | `cloud_storage_google` |  |  |
| `_migrate_local_to_cloud_storage` | internal rule | self, session | `cloud_storage_migration` |  | Migrate attachment from local binary storage to cloud storage |
| `_cron_migrate_local_to_cloud_storage` | background operation | self | `cloud_storage_migration` |  | The Http server only reschedules the cron job asap without migrating any attachment. The cron server will continue the migrating process stopped at the last time by using ``cloud_storage_migration_min_attachment_id`` |
| `_prevent_delete_from_submitted_expense` | internal rule | self | `hr_expense` | ondelete |  |
| `action_preview_attachment` | user action | self | `hr_fleet` |  |  |
| `init` | lifecycle override | self | `hr_recruitment` |  |  |
| `_l10n_fr_pdp_no_delete_fro_sent_flow` | internal rule | self | `l10n_fr_pdp` | ondelete |  |
| `_unlink_except_l10n_in_government_document` | internal rule | self | `l10n_in_edi` | ondelete | Prevents the deletion of attachments related to government-issued documents. |
| `_unlink_except_ewaybill_government_document` | internal rule | self | `l10n_in_ewaybill` | ondelete | Prevents the deletion of attachments related to government-issued documents. |
| `_except_submitted_invoices_pdfs` | internal rule | self | `l10n_jo_edi` | ondelete |  |
| `_unlink_except_posted_pdf_invoices` | internal rule | self | `l10n_sa` | ondelete | Prevents unlinking of invoice pdfs linked to an invoice that is posted. |
| `_get_posted_pdf_moves_to_check` | preparation rule | self | `l10n_sa_edi`, `l10n_sa` |  | Returns the moves to check whether they can be unlinked. |
| `_unlink_except_rejected_zatca_document` | internal rule | self | `l10n_sa_edi` | ondelete | Prevents unlinking of rejected XML documents |
| `_unlink_except_validated_pdf_invoices` | internal rule | self | `l10n_sa_edi` | ondelete | Prevents unlinking of invoice pdfs linked to an invoice where the pdf attachment was created after or at the same time as the edi_documents last write date. |

## Validation and error messages (31)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `force_storage` | AccessError | Only administrators can execute this action. | `base` |
| `_get_path` | UserError | The attachment collides with an existing file. | `base` |
| `_check_serving_attachments` | ValidationError | Sorry, you are not allowed to write on this document | `base` |
| `_check_circular_attachment` | ValidationError | You cannot attach an attachment to itself. Attachment %(record)s cannot have res_id: %(res_id)s | `base` |
| `check` | AccessError | Sorry, you are not allowed to access this document. | `base` |
| `check` | AccessError | Sorry, you are not allowed to access this document. | `base` |
| `write` | AccessError | Sorry, you are not allowed to access this document. | `base` |
| `create` | AccessError | Sorry, you are not allowed to access this document. | `base` |
| `create_unique` | UserError | Attachment is not encoded in base64. | `base` |
| `_can_return_content` | AccessError | Invalid access token | `base` |
| `_migrate_remote_to_local` | ValidationError | URL attachment (%s) shouldn't be migrated to local. | `base` |
| `_has_attachments_ownership` | UserError | An access token must be provided for each attachment. | `mail` |
| `_unlink_except_government_document` | UserError | You can't unlink an attachment being an EDI document sent to the government. | `account_edi` |
| `_post_add_create` | UserError | Cloud Storage is not enabled | `cloud_storage` |
| `_migrate_remote_to_local` | ValidationError | Failed to download attachment (%(id)s) from cloud: %(code)s - %(reason)s | `cloud_storage` |
| `_get_cloud_storage_azure_info` | ValidationError | %s is not a valid Azure Blob Storage URL. | `cloud_storage_azure` |
| `_get_cloud_storage_google_info` | ValidationError | %s is not a valid Google Cloud Storage URL. | `cloud_storage_google` |
| `_migrate_local_to_cloud_storage` | ValidationError | Attachment (%s) is not a binary attachment and cannot be migrated to cloud storage. | `cloud_storage_migration` |
| `_migrate_local_to_cloud_storage` | ValidationError | Attachment (%s) does not have a stored filename and cannot be migrated to cloud storage. | `cloud_storage_migration` |
| `_migrate_local_to_cloud_storage` | ValidationError | Failed to upload attachment %(id)s to cloud storage: %(code)s | `cloud_storage_migration` |
| `_cron_migrate_local_to_cloud_storage` | UserError | Cloud storage provider is not configured | `cloud_storage_migration` |
| `_cron_migrate_local_to_cloud_storage` | UserError | No model for cloud storage migration | `cloud_storage_migration` |
| `_prevent_delete_from_submitted_expense` | AccessError | You can't delete attachments from an expense once it has been submitted. | `hr_expense` |
| `create` | AccessError | You can't add attachments to an expense once it has been approved. | `hr_expense` |
| `_l10n_fr_pdp_no_delete_fro_sent_flow` | UserError | You can't delete an attachment linked to a sent Flow. | `l10n_fr_pdp` |
| `_unlink_except_l10n_in_government_document` | UserError | You can't unlink an attachment that you received from the government | `l10n_in_edi` |
| `_unlink_except_ewaybill_government_document` | UserError | You can't unlink an attachment that you received from the government | `l10n_in_ewaybill` |
| `_except_submitted_invoices_pdfs` | UserError | You cannot delete this Invoice PDF as it has been submitted to JoFotara | `l10n_jo_edi` |
| `_unlink_except_posted_pdf_invoices` | UserError | The Invoice PDF(s) cannot be deleted according to ZATCA rules: %s | `l10n_sa` |
| `_unlink_except_rejected_zatca_document` | UserError | You can't unlink an attachment being an EDI document refused by the government. | `l10n_sa_edi` |
| `_unlink_except_validated_pdf_invoices` | UserError | Oops! The invoice PDF(s) are linked to a validated EDI document and cannot be deleted according to ZATCA rules: %s | `l10n_sa_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | yes | yes | yes | yes | `base` |
| all internal users | no | no | no | no | `base` |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_attachment_form` | form |  | `name`, `type`, `datas`, `url`, `mimetype`, `res_model`, `res_field`, `res_id`, `res_name`, `company_id`, `public`, `create_uid`, `create_date`, `description`, `index_content` |  |  | `base` |
| `base.view_attachment_tree` | list |  | `name`, `res_model`, `res_field`, `res_id`, `type`, `file_size`, `company_id`, `create_uid`, `create_date` |  |  | `base` |
| `base.view_attachment_search` | search |  | `name`, `create_date`, `create_uid`, `type` |  | `My Document(s)`, `URL`, `Stored`, `Owner`, `Type`, `Company`, `Creation Date` | `base` |
| `hr_fleet.view_attachment_kanban_inherit_hr` | xpath | `mail.view_document_file_kanban` |  |  |  | `hr_fleet` |
| `hr_recruitment.ir_attachment_view_search_inherit_hr_recruitment` | xpath | `base.view_attachment_search` | `index_content` |  |  | `hr_recruitment` |
| `hr_recruitment.ir_attachment_hr_recruitment_list_view` | list |  | `name`, `res_id`, `res_model`, `datas`, `res_name`, `create_date` |  |  | `hr_recruitment` |
| `mail.view_document_file_kanban` | kanban |  | `id`, `mimetype`, `type`, `name`, `url`, `create_date`, `create_uid` |  |  | `mail` |
| `website.view_attachment_form_inherit_website` | field | `base.view_attachment_form` | `mimetype`, `website_id` |  |  | `website` |
| `website.view_attachment_tree_inherit_website` | field | `base.view_attachment_tree` | `name`, `website_id` |  |  | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_attachment` | Attachments |  |  |  |  | `base` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `cloud_storage_migration.ir_cron_manual_migrate_local_to_cloud_storage` | Migrate Local Attachment Binaries to Cloud Storage | 9999 months | `_cron_migrate_local_to_cloud_storage` |  |

Machine-readable definition: `../../../schemas/data/entities/ir.attachment.json`; views: `../../../schemas/interfaces/views/ir.attachment.json`.
