# Account electronic data interchange proxy user (`account_edi_proxy_client.user`)

**Transport name:** `account_edi_proxy_client.user`  
**Storage name:** `account_edi_proxy_client_user`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account_edi_proxy_client`  
**Extended by packages:** `account_peppol`, `account_peppol_response`, `l10n_dk_nemhandel`, `l10n_dk_nemhandel_response`, `l10n_fr_pdp`, `l10n_gr_edi_e_invoo`, `l10n_it_edi`, `l10n_my_edi`

Description: Account EDI proxy user

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `id_client` | Identifier Client | single line text |  | required |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company); indexed |
| `edi_identification` | Electronic data interchange Identification | single line text |  | required; Help: The unique id that identifies this user, typically the vat |
| `private_key_id` | Private Key | many to one | `certificate.key` | required; restricted by domain `[["public", "=", false]]`; Help: The key to encrypt all the user's data |
| `refresh_token` | Refresh Token | single line text |  | visible only to groups `base.group_system` |
| `is_token_out_of_sync` | Token Out of Sync | boolean |  | Help: This field is used to indicate that the edi user token is out of sync with the proxy server. It is set to True when the token needs to be refreshed or updated. |
| `token_sync_version` | Token Sync Version | integer |  |  |
| `proxy_type` | Proxy Type | selection |  | required; on delete of the target: {"l10n_my_edi": "cascade"}; extended by packages `account_peppol`, `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_gr_edi_e_invoo`, `l10n_it_edi`, `l10n_my_edi` |
| `edi_mode` | electronic data interchange operating mode | selection |  |  |
| `nemhandel_verification_code` | Nemhandel text message verification code | single line text |  |  |

## Selection values

### `proxy_type` (Proxy Type)

| Value | Label |
|---|---|
| `peppol` | PEPPOL |
| `nemhandel` | Nemhandel |
| `pdp` | Approved Platform |
| `l10n_it_edi` | Italian EDI |
| `l10n_my_edi` | Malaysian EDI |

### `edi_mode` (electronic data interchange operating mode)

| Value | Label |
|---|---|
| `prod` | Production mode |
| `test` | Test mode |
| `demo` | Demo mode |

## Database constraints and indexes (6)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_id_client` | Constraint | `unique(id_client)` | This id_client is already used on another user. | `account_edi_proxy_client` |
| `_unique_active_company_proxy` | UniqueIndex | `(company_id, proxy_type, edi_mode) WHERE (active IS TRUE)` | This company has an active user already created for this EDI type | `account_edi_proxy_client` |
| `_peppol_proxy_types_conflict` | Constraint | `EXCLUDE (                 company_id WITH =,                 edi_mode WITH =             )             WHERE (active IS TRUE AND proxy_type IN ('peppol', 'pdp'))` | You can not have both a Peppol and a PDP proxy user | `l10n_fr_pdp` |
| `_unique_identification_l10n_gr_edi` | UniqueIndex | `(edi_identification, edi_mode) WHERE (active IS TRUE AND proxy_type = 'l10n_gr_edi')` | This EDI identification is already assigned to an active user. | `l10n_gr_edi_e_invoo` |
| `_unique_identification_l10n_it_edi` | UniqueIndex | `(edi_identification, proxy_type, edi_mode) WHERE (active AND proxy_type = 'l10n_it_edi')` | This edi identification is already assigned to an active user | `l10n_it_edi` |
| `_unique_identification_l10n_my_edi` | UniqueIndex | `(edi_identification, proxy_type, edi_mode) WHERE (active AND proxy_type = 'l10n_my_edi')` | This edi identification is already assigned to an active user | `l10n_my_edi` |

## Operations (97)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_proxy_urls` | preparation rule | self | `account_edi_proxy_client`, `account_peppol`, `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_gr_edi_e_invoo`, `l10n_it_edi`, `l10n_my_edi` |  |  |
| `_get_server_url` | preparation rule | self, proxy_type, edi_mode | `account_edi_proxy_client` |  |  |
| `_get_proxy_users` | preparation rule | self, company, proxy_type | `account_edi_proxy_client` |  | Returns proxy users associated with the given company and proxy type. |
| `_get_proxy_identification` | preparation rule | self, company, proxy_type | `account_edi_proxy_client`, `account_peppol`, `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_gr_edi_e_invoo`, `l10n_it_edi`, `l10n_my_edi` |  | Returns the key that will identify company uniquely within a specific proxy type and edi operating mode. or raises a UserError (if the user didn't fill the related field). TO OVERRIDE |
| `_make_request` | internal rule | self, url, params, auth_type | `account_edi_proxy_client` |  | Make a request to proxy and handle the generic elements of the reponse (errors, new refresh token). |
| `_get_iap_params` | preparation rule | self, company, proxy_type, private_key_sudo | `account_edi_proxy_client`, `l10n_it_edi` |  |  |
| `_register_proxy_user` | internal rule | self, company, proxy_type, edi_mode | `account_edi_proxy_client`, `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_it_edi` |  | Generate the public_key/private_key that will be used to encrypt the file, send a request to the proxy to register the user with the public key and create the user with the private key.  :param company: the company of the user. |
| `_renew_token` | internal rule | self | `account_edi_proxy_client` |  | Request the proxy for a new refresh token.  Request to the proxy should be made with a refresh token that expire after 24h to avoid that multiple database use the same credentials. When receiving an error for an expired refresh_token, This method makes a request to get a new refresh token. |
| `_decrypt_data` | internal rule | self, data, symmetric_key | `account_edi_proxy_client` |  | Decrypt the data. Note that the data is encrypted with a symmetric key, which is encrypted with an asymmetric key. We must therefore decrypt the symmetric key.  :param data:            The data to decrypt. :param symmetric_key:   The symmetric_key encrypted with self.private_key_id.public_key() |
| `_get_peppol_proxy_types` | preparation rule | self | `account_peppol`, `l10n_fr_pdp` | model |  |
| `_get_peppol_proxy_endpoint` | preparation rule | self, endpoint, proxy_type | `account_peppol` |  | The `endpoint` should include the number; be like `2/participant_status` |
| `_get_peppol_error_message` | preparation rule | self, error_vals | `account_peppol` | model |  |
| `_call_peppol_proxy` | internal rule | self, endpoint, params | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_mark_connection_out_of_sync` | internal rule | self | `account_peppol` |  |  |
| `_peppol_out_of_sync_reconnect_this_database` | internal rule | self | `account_peppol` |  |  |
| `_peppol_out_of_sync_disconnect_this_database` | internal rule | self | `account_peppol` |  |  |
| `_get_can_send_domain` | preparation rule | self | `account_peppol` | model |  |
| `_cron_peppol_get_new_documents` | background operation | self | `account_peppol` |  |  |
| `_cron_peppol_get_message_status` | background operation | self | `account_peppol` |  |  |
| `_cron_peppol_get_participant_status` | background operation | self | `account_peppol` |  |  |
| `_cron_peppol_webhook_keepalive` | background operation | self | `account_peppol` |  |  |
| `_get_type_code` | preparation rule | self, files_data | `account_peppol`, `l10n_fr_pdp` |  | Override to unwrap the XML from the PDF with Factur-X. |
| `_peppol_import_invoice` | internal rule | self, attachment, peppol_state, uuid, journal | `account_peppol` |  | Save new documents in an accounting journal, when one is specified on the company.  :param attachment: the new document :param peppol_state: the state of the received Peppol document :param uuid: the UUID of the Peppol document :param journal: journal to use for the new move (otherwise the company's peppol journal will be used) :return: the created move (if any) |
| `_peppol_get_duplicate_message_uuids` | internal rule | self, message_uuids | `account_peppol_response`, `account_peppol` |  |  |
| `_peppol_get_new_documents` | internal rule | self, skip_no_journal | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_peppol_get_filetype` | internal rule | self, content | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_peppol_get_decoded_document` | internal rule | self, content | `account_peppol` |  |  |
| `_peppol_process_new_messages` | internal rule | self, messages | `account_peppol_response`, `account_peppol`, `l10n_fr_pdp` |  |  |
| `_peppol_post_process_new_messages` | internal rule | self, moves | `account_peppol_response`, `account_peppol` |  |  |
| `_peppol_get_message_status` | internal rule | self | `account_peppol` |  |  |
| `_peppol_get_documents_for_status` | internal rule | self, batch_size | `account_peppol_response`, `account_peppol` |  |  |
| `_peppol_process_messages_status` | internal rule | self, messages, uuid_to_record | `account_peppol_response`, `account_peppol`, `l10n_fr_pdp` |  |  |
| `_peppol_get_message_status_error_body` | internal rule | self, move, error | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_peppol_get_message_status_update_body` | internal rule | self, move, content | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_peppol_process_participant_status` | internal rule | self, proxy_user | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_peppol_get_participant_status` | internal rule | self | `account_peppol` |  |  |
| `_get_company_details` | preparation rule | self | `account_peppol` |  |  |
| `_peppol_register_sender` | internal rule | self, peppol_external_provider | `account_peppol` |  |  |
| `_peppol_register_sender_as_receiver` | internal rule | self | `account_peppol` |  |  |
| `_peppol_deregister_participant` | internal rule | self | `account_peppol` |  |  |
| `_peppol_deregister_participant_to_sender` | internal rule | self | `account_peppol` |  |  |
| `_peppol_auto_register_services` | internal rule | self, module | `account_peppol` | model |  |
| `_peppol_auto_deregister_services` | internal rule | self, module | `account_peppol` | model | Unregister a set of document types for all recipient users.  This function should be run in the uninstall hook of any module that extends the supported document types.  :param module: Module from which this function is being called, allows us to determine which     document types are no longer supported. |
| `_peppol_get_services` | internal rule | self | `account_peppol` |  | Get information from the IAP regarding the Peppol services. |
| `_generate_webhook_token` | internal rule | self, company | `account_peppol` | model |  |
| `_get_user_from_token` | preparation rule | self, token, url | `account_peppol` | model |  |
| `_peppol_reset_webhook` | internal rule | self | `account_peppol` |  |  |
| `_peppol_send_response` | internal rule | self, reference_moves, status, clarifications | `account_peppol_response` |  |  |
| `_peppol_extract_response_info` | internal rule | self, document | `account_peppol_response` | model |  |
| `_cron_peppol_auto_register_services` | background operation | self | `account_peppol_response` |  |  |
| `_call_nemhandel_proxy` | internal rule | self, endpoint, params | `l10n_dk_nemhandel` |  |  |
| `_check_user_on_alternative_service` | validation | self | `l10n_dk_nemhandel` |  |  |
| `_cron_nemhandel_get_new_documents` | background operation | self | `l10n_dk_nemhandel` |  |  |
| `_cron_nemhandel_get_message_status` | background operation | self | `l10n_dk_nemhandel` |  |  |
| `_cron_nemhandel_get_participant_status` | background operation | self | `l10n_dk_nemhandel` |  |  |
| `_cron_nemhandel_webhook_keepalive` | background operation | self | `l10n_dk_nemhandel` |  |  |
| `_nemhandel_import_invoice` | internal rule | self, attachment, nemhandel_state, uuid, journal | `l10n_dk_nemhandel` |  | Save new documents in an accounting journal, when one is specified on the company.  :param attachment: the new document :param nemhandel_state: the state of the received Nemhandel document :param uuid: the UUID of the Nemhandel document :return: the created invoice if the document was saved, `False` if it was not |
| `_nemhandel_get_new_documents` | internal rule | self, skip_no_journal, batch_size | `l10n_dk_nemhandel` |  |  |
| `_nemhandel_process_new_messages` | internal rule | self, messages | `l10n_dk_nemhandel_response`, `l10n_dk_nemhandel` |  |  |
| `_nemhandel_post_process_new_messages` | internal rule | self, moves | `l10n_dk_nemhandel` |  |  |
| `_nemhandel_get_message_status` | internal rule | self, batch_size | `l10n_dk_nemhandel` |  |  |
| `_nemhandel_get_documents_for_status` | internal rule | self, batch_size | `l10n_dk_nemhandel_response`, `l10n_dk_nemhandel` |  |  |
| `_nemhandel_process_messages_status` | internal rule | self, messages, uuid_to_record | `l10n_dk_nemhandel_response`, `l10n_dk_nemhandel` |  |  |
| `_nemhandel_get_participant_status` | internal rule | self | `l10n_dk_nemhandel` |  |  |
| `_get_nemhandel_company_details` | preparation rule | self | `l10n_dk_nemhandel` |  |  |
| `_nemhandel_register_as_receiver` | internal rule | self | `l10n_dk_nemhandel` |  |  |
| `_nemhandel_deregister_participant` | internal rule | self | `l10n_dk_nemhandel` |  |  |
| `_generate_nemhandel_webhook_token` | internal rule | self | `l10n_dk_nemhandel` |  |  |
| `_get_nemhandel_user_from_token` | preparation rule | self, token, url | `l10n_dk_nemhandel` | model |  |
| `_nemhandel_reset_webhook` | internal rule | self | `l10n_dk_nemhandel` |  |  |
| `_nemhandel_send_response` | internal rule | self, reference_moves, status, note | `l10n_dk_nemhandel_response` |  |  |
| `_nemhandel_extract_response_info` | internal rule | self, document | `l10n_dk_nemhandel_response` | model |  |
| `_pdp_register_receiver` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_pdp_get_regulatory_documents` | internal rule | self, batch_size | `l10n_fr_pdp` |  |  |
| `_pdp_send_response` | internal rule | self, reference_moves, status, additional_info | `l10n_fr_pdp` |  |  |
| `_pdp_process_regulatory_messages` | internal rule | self, messages | `l10n_fr_pdp` |  |  |
| `_pdp_import_flow_10_response` | internal rule | self, uuid, content | `l10n_fr_pdp` |  |  |
| `_pdp_import_tax_extract` | internal rule | self, uuid, content, origin_move | `l10n_fr_pdp` |  |  |
| `_pdp_import_outgoing_response` | internal rule | self, uuid, content, origin_move | `l10n_fr_pdp` |  |  |
| `_pdp_import_incoming_response` | internal rule | self, uuid, content, origin_move | `l10n_fr_pdp` |  |  |
| `_pdp_parse_included_note` | internal rule | self, note_node | `l10n_fr_pdp` | model |  |
| `_pdp_extract_response_info` | internal rule | self, document | `l10n_fr_pdp` | model |  |
| `_format_payment_info` | internal rule | self, info, separator | `l10n_fr_pdp` | model |  |
| `_format_status_info` | internal rule | self, status, separator | `l10n_fr_pdp` | model |  |
| `_pdp_status_infos_to_details` | internal rule | self, status_infos | `l10n_fr_pdp` | model |  |
| `_pdp_proxy_error_message` | internal rule | self, error | `l10n_fr_pdp` | model |  |
| `_pdp_selection_label` | internal rule | self, record, field_name, value | `l10n_fr_pdp` | model |  |
| `_pdp_format_multiline_value` | internal rule | self, value | `l10n_fr_pdp` | model |  |
| `_pdp_log_einvoicing_chatter` | internal rule | self, move, pa_status, ppf_status, details, errors, error_source | `l10n_fr_pdp` | model |  |
| `_pdp_format_einvoicing_message` | internal rule | self, pa_status, ppf_status, details, errors, error_source | `l10n_fr_pdp` | model |  |
| `_pdp_format_message_body` | internal rule | self, flow_number, main_message, markup_status_info | `l10n_fr_pdp` | model |  |
| `_pdp_send_lifecycles` | internal rule | self, batch_size | `l10n_fr_pdp` |  |  |
| `_cron_pdp_get_regulatory_documents` | background operation | self | `l10n_fr_pdp` |  |  |
| `_cron_pdp_send_lifecycles` | background operation | self | `l10n_fr_pdp` |  |  |
| `_l10n_gr_edi_proxy_request` | internal rule | self, route, params | `l10n_gr_edi_e_invoo` |  |  |
| `_toggle_proxy_user_active` | internal rule | self | `l10n_it_edi` |  | Toggle the value of the ``active`` boolean field of the proxy_user, and handle sending the reactivate/deactivate requests to the IAP side. |
| `_l10n_my_edi_contact_proxy` | internal rule | self, endpoint, params | `l10n_my_edi` |  |  |

## Validation and error messages (35)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_register_proxy_user` | UserError | A user already exists with this identification. | `account_edi_proxy_client` |
| `_register_proxy_user` | UserError | response['error'] | `account_edi_proxy_client` |
| `_register_proxy_user` | UserError | e.message | `account_edi_proxy_client` |
| `_register_proxy_user` | UserError | A user already exists with theses credentials on our server. Please check your information. | `account_edi_proxy_client` |
| `_call_peppol_proxy` | UserError | EDI user should be of one of the following types: %s | `account_peppol` |
| `_call_peppol_proxy` | UserError | token_out_of_sync_error_message | `account_peppol` |
| `_call_peppol_proxy` | UserError | error_message | `account_peppol` |
| `_call_peppol_proxy` | UserError | e.message | `account_peppol` |
| `_call_peppol_proxy` | UserError | We could not find a user with this information on our server. Please check your information. | `account_peppol` |
| `_call_peppol_proxy` | UserError | token_out_of_sync_error_message | `account_peppol` |
| `_mark_connection_out_of_sync` | UserError | This connection has been superseded by another database. Register again. | `account_peppol` |
| `_get_proxy_identification` | UserError | Please fill in the EAS code and the Participant ID code. | `account_peppol` |
| `_peppol_get_new_documents` | UserError | msg | `account_peppol` |
| `_peppol_register_sender_as_receiver` | UserError | Cannot register a user with a %s application | `account_peppol` |
| `_peppol_register_sender_as_receiver` | UserError | error_msg | `account_peppol` |
| `_peppol_send_response` | ValidationError | At least one reason must be given when rejecting a Peppol invoice. | `account_peppol_response` |
| `_call_nemhandel_proxy` | UserError | EDI user should be of type Nemhandel | `l10n_dk_nemhandel` |
| `_call_nemhandel_proxy` | UserError | errors.get(error_code) or error_message or _('Connection error, please try again later.') | `l10n_dk_nemhandel` |
| `_call_nemhandel_proxy` | UserError | e.message | `l10n_dk_nemhandel` |
| `_check_user_on_alternative_service` | UserError | error_msg | `l10n_dk_nemhandel` |
| `_get_proxy_identification` | UserError | Please fill in the Identifier Type and Value. | `l10n_dk_nemhandel` |
| `_register_proxy_user` | UserError | e.message | `l10n_dk_nemhandel` |
| `_nemhandel_get_new_documents` | UserError | msg | `l10n_dk_nemhandel` |
| `_nemhandel_register_as_receiver` | UserError | Cannot register a user with a %s application | `l10n_dk_nemhandel` |
| `_nemhandel_register_as_receiver` | ValidationError | If you try to register with your CVR, please make sure your company has the same VAT | `l10n_dk_nemhandel` |
| `_get_proxy_identification` | UserError | Please fill the Peppol Endpoint field with scheme '%s' on the company partner. | `l10n_fr_pdp` |
| `_register_proxy_user` | UserError | error_message | `l10n_fr_pdp` |
| `_register_proxy_user` | UserError | e.message | `l10n_fr_pdp` |
| `_pdp_register_receiver` | UserError | This is only possible for the 'Approved Platform'. | `l10n_fr_pdp` |
| `_pdp_register_receiver` | UserError | Cannot register a user with a '%(proxy_state)s' application. | `l10n_fr_pdp` |
| `_pdp_send_response` | UserError | Unsupported response status: '%s'. | `l10n_fr_pdp` |
| `_get_proxy_identification` | UserError | Please fill your codice fiscale to be able to receive invoices from FatturaPA | `l10n_it_edi` |
| `_get_proxy_identification` | UserError | Please fill the TIN of company "%(company_name)s" before enabling the integration with MyInvois. | `l10n_my_edi` |
| `_l10n_my_edi_contact_proxy` | UserError | The MyInvois server is temporarily unreachable. Please try again later. | `l10n_my_edi` |
| `_l10n_my_edi_contact_proxy` | UserError | You have reached the maximum number of requests allowed in a short period of time. Please wait a few minutes before trying again. | `l10n_my_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `account_edi_proxy_client` |
| `account.group_account_invoice` | no | yes | no | no | `account_edi_proxy_client` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account EDI Proxy Client User | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_edi_proxy_client.view_form_account_edi_proxy_client_user` | form |  | `company_id`, `proxy_type`, `edi_mode`, `id_client`, `edi_identification`, `private_key_id`, `refresh_token` |  |  | `account_edi_proxy_client` |
| `account_edi_proxy_client.view_tree_account_edi_proxy_client_user` | list |  | `company_id`, `proxy_type`, `edi_mode`, `id_client`, `edi_identification`, `private_key_id`, `refresh_token` |  |  | `account_edi_proxy_client` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account_edi_proxy_client.action_tree_account_edi_proxy_client_user` | EDI Proxy User | list,form |  |  |  | `account_edi_proxy_client` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `account_peppol.ir_cron_peppol_get_new_documents` | PEPPOL: retrieve new documents | 4 hours | `_cron_peppol_get_new_documents` |  |
| `account_peppol.ir_cron_peppol_get_message_status` | PEPPOL: update message status | 1 days | `_cron_peppol_get_message_status` |  |
| `account_peppol.ir_cron_peppol_get_participant_status` | PEPPOL: update participant status | 1 weeks | `_cron_peppol_get_participant_status` |  |
| `account_peppol.ir_cron_peppol_webhook_keepalive` | PEPPOL: webhook keep alive | 2 weeks | `_cron_peppol_webhook_keepalive` |  |
| `account_peppol_response.ir_cron_peppol_auto_register_services` | PEPPOL: auto register services | 999 months | `_cron_peppol_auto_register_services` |  |
| `l10n_dk_nemhandel.ir_cron_nemhandel_get_new_documents` | Nemhandel: retrieve new documents | 4 hours | `_cron_nemhandel_get_new_documents` |  |
| `l10n_dk_nemhandel.ir_cron_nemhandel_get_message_status` | Nemhandel: update message status | 1 days | `_cron_nemhandel_get_message_status` |  |
| `l10n_dk_nemhandel.ir_cron_nemhandel_webhook_keepalive` | Nemhandel: webhook keep alive | 2 weeks | `_cron_nemhandel_webhook_keepalive` |  |
| `l10n_dk_nemhandel.ir_cron_nemhandel_get_participant_status` | Nemhandel: update participant status | 1 weeks | `_cron_nemhandel_get_participant_status` |  |
| `l10n_fr_pdp.ir_cron_pdp_get_regulatory_documents` | PDP: Retrieve new regulatory documents | 4 hours | `_cron_pdp_get_regulatory_documents` |  |
| `l10n_fr_pdp.ir_cron_pdp_send_lifecycles` | PDP: Send lifecycles | 12 hours | `_cron_pdp_send_lifecycles` |  |

Machine-readable definition: `../../../schemas/data/entities/account_edi_proxy_client.user.json`; views: `../../../schemas/interfaces/views/account_edi_proxy_client.user.json`.
