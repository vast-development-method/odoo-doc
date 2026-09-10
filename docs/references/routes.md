# Routes

Every route exposed over Hypertext Transfer Protocol.

| Path | Request type | Authentication | Methods | Website | Package | Operation | Purpose |
|---|---|---|---|---|---|---|---|
| `/product/catalog/get_sections` | jsonrpc | user |  |  | `account` | `ProductCatalogAccountController.product_catalog_get_sections` | Return the sections which are in given order to be shown in the product catalog.  :param string res_model: The order model. :param int order_id: The order id. :param string child_field: The field name of the lines in the order model. :rtype: list :return: A list of dictionaries containing section in |
| `/product/catalog/create_section` | jsonrpc | user |  |  | `account` | `ProductCatalogAccountController.product_catalog_create_section` | Create a new section on the given order.  :param string res_model: The order model. :param int order_id: The order id. :param string child_field: The field name of the lines in the order model. :param string name: The name of the section to create. :param str position: The position of the section wh |
| `/product/catalog/resequence_sections` | jsonrpc | user |  |  | `account` | `ProductCatalogAccountController.product_catalog_resequence_sections` | Reorder the sections of a given order.  param string res_model: The order model. :param int order_id: The order id. :param list sections:  A list of section dictionaries with their sequence. :param string child_field: The field name of the lines in the order model. :return: A dictionary with new seq |
| `/account/download_invoice_attachments/<models("ir.attachment"):attachments>` | http | user |  |  | `account` | `AccountDocumentDownloadController.download_invoice_attachments` |  |
| `/account/download_invoice_documents/<models("account.move"):invoices>/<string:filetype>` | http | user |  |  | `account` | `AccountDocumentDownloadController.download_invoice_documents_filetype` |  |
| `/account/download_move_attachments/<models("account.move"):moves>` | http | user |  |  | `account` | `AccountDocumentDownloadController.download_move_attachments` |  |
| `/my/invoices` | http | user |  | True | `account` | `PortalAccount.portal_my_invoices` |  |
| `/my/invoices/page/<int:page>` | http | user |  | True | `account` | `PortalAccount.portal_my_invoices` |  |
| `/my/invoices/<int:invoice_id>` | http | public |  | True | `account` | `PortalAccount.portal_my_invoice_detail` |  |
| `/my/journal/<int:journal_id>/unsubscribe` | http | public | ["GET", "POST"] | True | `account` | `PortalAccount.portal_my_journal_unsubscribe` |  |
| `/terms` | http | public |  | True | `account` | `TermsController.terms_conditions` |  |
| `/account/init_tests_shared_js_python` | http | user |  | True | `account` | `TestsSharedJsPython.route_init_tests_shared_js_python` |  |
| `/account/post_tests_shared_js_python` | jsonrpc | user |  |  | `account` | `TestsSharedJsPython.route_post_tests_shared_js_python` |  |
| `/invoice/transaction/<int:invoice_id>` | jsonrpc | public |  |  | `account_payment` | `PaymentPortal.invoice_transaction` | Create a draft transaction and return its processing values.  :param int invoice_id: The invoice to pay, as an `account.move` id :param str access_token: The access token used to authenticate the request :param dict kwargs: Locally unused data passed to `_create_transaction` :return: The mandatory v |
| `/invoice/transaction/overdue` | jsonrpc | public |  |  | `account_payment` | `PaymentPortal.overdue_invoices_transaction` | Create a draft transaction for overdue invoices and return its processing values.  :param str payment_reference: The reference to the current payment :param dict kwargs: Locally unused data passed to `_create_transaction` :return: The mandatory values for the processing of the transaction :rtype: di |
| `/my/invoices/overdue` | http | public | ["GET"] | True | `account_payment` | `PortalAccount.portal_my_overdue_invoices` |  |
| `/peppol/authentication/callback` | http | user | ["GET"] |  | `account_peppol` | `PeppolAuthentication.peppol_authentication_callback` | Route called by the Proxy Server after authentication. |
| `/peppol/authentication/webhook` | http | public | ["POST"] |  | `account_peppol` | `PeppolAuthentication.peppol_authentication_webhook` | webhook called by IAP on positive KYC decision.  Finalizes the registration automatically |
| `/peppol/webhook/new-message` | http | public | ["POST"] |  | `account_peppol` | `PeppolWebhookController.webhook_new_message` |  |
| `/peppol/webhook/message-state-update` | http | public | ["POST"] |  | `account_peppol` | `PeppolWebhookController.webhook_message_update` |  |
| `/peppol/webhook/user-state-update` | http | public | ["POST"] |  | `account_peppol` | `PeppolWebhookController.webhook_user_update` |  |
| `/doc` | http | user |  |  | `api_doc` | `DocController.doc_client` |  |
| `/doc/<model_name>` | http | user |  |  | `api_doc` | `DocController.doc_client` |  |
| `/doc/index.html` | http | user |  |  | `api_doc` | `DocController.doc_client` |  |
| `/doc-bearer/index.json` | json2 | bearer |  |  | `api_doc` | `DocController.doc_bearer_index` |  |
| `/doc/index.json` | json2 | user |  |  | `api_doc` | `DocController.doc_index` | Get a listing of all modules, models, methods and fields. But only their technical name and translated "human" name.  It returns a json-serialized dictionnary with the following structure:  .. code-block:: python     {         'modules': list[str],         'models': [             {                 ' |
| `/doc-bearer/<model_name>.json` | json2 | bearer |  |  | `api_doc` | `DocController.doc_bearer_modec` |  |
| `/doc/<model_name>.json` | json2 | user |  |  | `api_doc` | `DocController.doc_model` | Get a complete listing of all the methods and fields for a specific model. The listing includes the htmlified docstring of the model, an enriched fields_get(), the methods signature, parameters and htmlified docstrings.  It returns a json-serialized dictionnary with the following structure:  .. code |
| `/auth_oauth/signin` | http | none |  |  | `auth_oauth` | `OAuthController.signin` |  |
| `/auth_oauth/oea` | http | none |  |  | `auth_oauth` | `OAuthController.oea` | login user via the system Account provider |
| `/auth/passkey/start-auth` | jsonrpc | public |  |  | `auth_passkey` | `WebauthnController.json_start_authentication` |  |
| `/.well-known/assetlinks.json` | http | public |  |  | `auth_passkey` | `WebauthnController.web_well_known_android` |  |
| `/web/signup` | http | public |  | True | `auth_signup` | `AuthSignupHome.web_auth_signup` |  |
| `/web/reset_password` | http | public |  | True | `auth_signup` | `AuthSignupHome.web_auth_reset_password` |  |
| `/.well-known/change-password` | http | public | ["GET"] |  | `auth_signup` | `AuthSignupHome.well_known_change_password` | Redirect to the password reset form.  Implements the Change Password URL specification: https://wicg.github.io/change-password-url/ |
| `/auth-timeout/check-identity` | http | user |  | True | `auth_timeout` | `AuthTimeOutController.check_identity` | Display the authentication form in a page. Used when an HTTP call raises a `CheckIdentityException`. |
| `/auth-timeout/session/check-identity` | jsonrpc | user |  |  | `auth_timeout` | `AuthTimeOutController.check_identity_session` | JSON route used to receive the authentication form sent by the user. |
| `/auth-timeout/send-totp-mail-code` | jsonrpc | user |  |  | `auth_timeout` | `AuthTimeOutController.send_totp_mail_code` | JSON route to trigger the sending of the TOTP code by email when requested by the user in the interface. |
| `/web/login/totp` | http | public | ["GET", "POST"] | True | `auth_totp` | `Home.web_totp` |  |
| `/web/hook/<string:rule_uuid>` | http | public | ["GET", "POST"] |  | `base_automation` | `BaseAutomationController.call_webhook_http` | Execute an automation webhook |
| `/base_import/set_file` | http | user | ["POST"] |  | `base_import` | `ImportController.set_file` |  |
| `/base_import_module/login_upload` | http | none | ["POST"] |  | `base_import_module` | `ImportModule.login_upload` |  |
| `/kpi/summary` | jsonrpc | none |  |  | `base_setup` | `KpiController.kpi_summary` | Retrieve the KPI summaries from a batch of databases hosted on the same server. The result of this call will only include the databases:     - that have been found on this server     - where the provided API key could be verified     - that are on the same the system version as the current the syste |
| `/base_setup/data` | jsonrpc | user |  |  | `base_setup` | `BaseSetup.base_setup_data` |  |
| `/base_setup/demo_active` | jsonrpc | user |  |  | `base_setup` | `BaseSetup.base_setup_is_demo` |  |
| `/base_vat/1/webhook_update_vies` | http | public |  |  | `base_vat` | `BaseVatWebhookController.webhook_update_vies` | Webhook called by IAP when it updates a status from the pending state. The webhook_token is computed by the the system db (in _compute_vies_valid) and stored on IAP such that only IAP can call this webhook. |
| `/board/add_to_dashboard` | jsonrpc | user |  |  | `board` | `Board.add_to_dashboard` |  |
| `/bus/get_model_definitions` | http | user | ["POST"] |  | `bus` | `BusController.get_model_definitions` |  |
| `/bus/has_missed_notifications` | jsonrpc | public |  |  | `bus` | `BusController.has_missed_notifications` |  |
| `/websocket` | http | public |  |  | `bus` | `WebsocketController.websocket` | Handle the websocket handshake, upgrade the connection if successfull.  :param version: The version of the WebSocket worker that tries to     connect. Connections with an outdated version will result in the     websocket being closed. See :attr:`WebsocketConnectionHandler._VERSION`. |
| `/websocket/health` | http | none |  |  | `bus` | `WebsocketController.health` |  |
| `/websocket/peek_notifications` | jsonrpc | public |  |  | `bus` | `WebsocketController.peek_notifications` |  |
| `/websocket/on_closed` | jsonrpc | public |  |  | `bus` | `WebsocketController.on_websocket_closed` | Manually notify the closure of a websocket, useful when implementing custom websocket code. This is mainly used by the system.sh. |
| `/bus/websocket_worker_bundle` | http | public |  |  | `bus` | `WebsocketController.get_websocket_worker_bundle` | :param str v: Version of the worker, frontend only argument used to     prevent new worker versions to be loaded from the browser cache. |
| `/calendar/meeting/accept` | http | calendar |  |  | `calendar` | `CalendarController.accept_meeting` |  |
| `/calendar/recurrence/accept` | http | calendar |  |  | `calendar` | `CalendarController.accept_recurrence` |  |
| `/calendar/meeting/decline` | http | calendar |  |  | `calendar` | `CalendarController.decline_meeting` |  |
| `/calendar/recurrence/decline` | http | calendar |  |  | `calendar` | `CalendarController.decline_recurrence` |  |
| `/calendar/meeting/view` | http | calendar |  |  | `calendar` | `CalendarController.view_meeting` |  |
| `/calendar/meeting/join` | http | user |  | True | `calendar` | `CalendarController.calendar_join_meeting` |  |
| `/calendar/notify` | jsonrpc | user |  |  | `calendar` | `CalendarController.notify` |  |
| `/calendar/notify_ack` | jsonrpc | user |  |  | `calendar` | `CalendarController.notify_ack` |  |
| `/calendar/join_videocall/<string:access_token>` | http | public |  |  | `calendar` | `CalendarController.calendar_join_videocall` |  |
| `/calendar/check_credentials` | jsonrpc | user |  |  | `calendar` | `CalendarController.check_calendar_credentials` |  |
| `/lead/case_mark_won` | http | user | ["GET"] |  | `crm` | `CrmController.crm_lead_case_mark_won` |  |
| `/lead/case_mark_lost` | http | user | ["GET"] |  | `crm` | `CrmController.crm_lead_case_mark_lost` |  |
| `/lead/convert` | http | user | ["GET"] |  | `crm` | `CrmController.crm_lead_convert` |  |
| `/mail_client_extension/log_single_mail_content` | jsonrpc | outlook |  |  | `crm_mail_plugin` | `CrmClient.log_single_mail_content` | deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary for supporting older versions |
| `/mail_client_extension/lead/get_by_partner_id` | jsonrpc | outlook |  |  | `crm_mail_plugin` | `CrmClient.crm_lead_get_by_partner_id` | deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary for supporting older versions |
| `/mail_client_extension/lead/create_from_partner` | http | user | ["GET"] |  | `crm_mail_plugin` | `CrmClient.crm_lead_redirect_create_form_view` | deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary for supporting older versions |
| `/mail_plugin/lead/create` | jsonrpc | outlook |  |  | `crm_mail_plugin` | `CrmClient.crm_lead_create` |  |
| `/mail_client_extension/lead/open` | http | user |  |  | `crm_mail_plugin` | `CrmClient.crm_lead_open` | deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary for supporting older versions |
| `/delivery/set_pickup_location` | jsonrpc | user |  |  | `delivery` | `LocationSelectorController.delivery_set_pickup_location` | Fetch the order and set the pickup location on the current order.  :param int order_id: The sales order, as a `sale.order` id. :param str pickup_location_data: The JSON-formatted pickup location address. :return: None |
| `/delivery/get_pickup_locations` | jsonrpc | user |  |  | `delivery` | `LocationSelectorController.delivery_get_pickup_locations` | Fetch the order and return the pickup locations close to a given zip code.  Determine the country based on GeoIP or fallback on the order's delivery address' country.  :param int order_id: The sales order, as a `sale.order` id. :param int zip_code: The zip code to look up to. :return: The close pick |
| `/digest/<int:digest_id>/unsubscribe_oneclik` | http | public | ["POST"] | True | `digest` | `DigestController.digest_unsubscribe_oneclick` | Propose a one click button to the user to unsubscribe as defined in Only POST method is allowed preventing the risk that anti-spam trigger unwanted unsubscribe (scenario explained in the same rfc). Note: this method must support encoding method 'multipart/form-data' and 'application/x-www-form-urlen |
| `/digest/<int:digest_id>/unsubscribe` | http | public | ["GET", "POST"] | True | `digest` | `DigestController.digest_unsubscribe` | Unsubscribe a given user from a given digest  :param int digest_id: id of digest to unsubscribe from :param str token: token preventing URL forgery :param user_id: id of user to unsubscribe  :param int one_click: set it to 1 when using the URL in the header of   the email to allow mail user agent to |
| `/digest/<int:digest_id>/set_periodicity` | http | user |  | True | `digest` | `DigestController.digest_set_periodicity` |  |
| `/event/<model("event.event"):event>/ics` | http | public |  | True | `event` | `EventController.event_ics_file` |  |
| `/event/<int:event_id>/my_tickets` | http | public |  |  | `event` | `EventController.event_my_tickets` | Returns a pdf response, containing all tickets for attendees in registration_ids for event_id.  Throw Forbidden if no registration is valid / hash is invalid / parameters are missing. This route is used in links in emails to attendees, as well as in registration confirmation screens.  :param event:  |
| `/event/init_barcode_interface` | jsonrpc | user |  |  | `event` | `EventController.init_barcode_interface` |  |
| `/google_account/authentication` | http | public |  |  | `google_account` | `GoogleAuth.oauth2callback` | This route/function is called by Google when user Accept/Refuse the consent of Google |
| `/autocomplete/address` | jsonrpc | public | ["POST"] | True | `google_address_autocomplete` | `AutoCompleteController._autocomplete_address` |  |
| `/autocomplete/address_full` | jsonrpc | public | ["POST"] | True | `google_address_autocomplete` | `AutoCompleteController._autocomplete_address_full` |  |
| `/google_calendar/sync_data` | jsonrpc | user |  |  | `google_calendar` | `GoogleCalendarController.google_calendar_sync_data` | This route/function is called when we want to synchronize the system calendar with Google Calendar. Function return a dictionary with the status :  need_config_from_admin, need_auth, need_refresh, sync_stopped, success if not calendar_event The dictionary may contains an url, to allow the system Cli |
| `/google_gmail/confirm` | http | user |  |  | `google_gmail` | `GoogleGmailController.google_gmail_callback` | Callback URL during the OAuth process.  Gmail redirects the user browser to this endpoint with the authorization code. We will fetch the refresh token and the access token thanks to this authorization code and save those values on the given mail server. |
| `/google_gmail/iap_confirm` | http | user |  |  | `google_gmail` | `GoogleGmailController.google_gmail_iap_callback` | Receive back the refresh token and access token from IAP.  The authentication process with IAP is done in 4 steps; 1. User database make a request to `<IAP>/api/mail_oauth/1/gmail` 2. User browser is redirected to the URL we received from IAP 3. User browser is redirected to `<IAP>/api/mail_oauth/1/ |
| `/hr_attendance/kiosk_mode_menu/<int:company_id>` | http | user |  |  | `hr_attendance` | `HrAttendance.kiosk_menu_item_action` |  |
| `/hr_attendance/get_employees_without_badge` | jsonrpc | public |  |  | `hr_attendance` | `HrAttendance.get_employees_without_badge` | Fetch only employees without a badge (barcode). |
| `/hr_attendance/set_badge` | jsonrpc | public |  |  | `hr_attendance` | `HrAttendance.set_badge` |  |
| `/hr_attendance/create_employee` | jsonrpc | public |  |  | `hr_attendance` | `HrAttendance.create_employee` |  |
| `/hr_attendance/kiosk_keepalive` | jsonrpc | user |  |  | `hr_attendance` | `HrAttendance.kiosk_keepalive` |  |
| `/hr_attendance/<token>` | http | public |  | True | `hr_attendance` | `HrAttendance.open_kiosk_mode` |  |
| `/hr_attendance/attendance_employee_data` | jsonrpc | public |  |  | `hr_attendance` | `HrAttendance.employee_attendance_data` |  |
| `/hr_attendance/attendance_barcode_scanned` | jsonrpc | public |  |  | `hr_attendance` | `HrAttendance.scan_barcode_with_geolocation` |  |
| `/hr_attendance/manual_selection` | jsonrpc | public |  |  | `hr_attendance` | `HrAttendance.manual_selection` |  |
| `/hr_attendance/employees_infos` | jsonrpc | public |  |  | `hr_attendance` | `HrAttendance.employees_infos` |  |
| `/hr_attendance/systray_check_in_out` | jsonrpc | user |  |  | `hr_attendance` | `HrAttendance.systray_attendance` |  |
| `/hr_attendance/attendance_user_data` | jsonrpc | user |  |  | `hr_attendance` | `HrAttendance.user_attendance_data` |  |
| `/hr_attendance/set_settings` | jsonrpc | public |  |  | `hr_attendance` | `HrAttendance.set_attendance_settings` |  |
| `/leave/approve` | http | user | ["GET"] |  | `hr_holidays` | `HrHolidaysController.hr_holidays_request_approve` |  |
| `/leave/validate` | http | user | ["GET"] |  | `hr_holidays` | `HrHolidaysController.hr_holidays_request_validate` |  |
| `/leave/refuse` | http | user | ["GET"] |  | `hr_holidays` | `HrHolidaysController.hr_holidays_request_refuse` |  |
| `/allocation/validate` | http | user | ["GET"] |  | `hr_holidays` | `HrHolidaysController.hr_holidays_allocation_validate` |  |
| `/allocation/refuse` | http | user | ["GET"] |  | `hr_holidays` | `HrHolidaysController.hr_holidays_allocation_refuse` |  |
| `/hr/get_redirect_model` | jsonrpc | user |  |  | `hr_org_chart` | `HrOrgChartController.get_redirect_model` |  |
| `/hr/get_org_chart` | jsonrpc | user |  |  | `hr_org_chart` | `HrOrgChartController.get_org_chart` |  |
| `/hr/get_subordinates` | jsonrpc | user |  |  | `hr_org_chart` | `HrOrgChartController.get_subordinates` | Get employee subordinates. Possible values for 'subordinates_type':     - 'indirect'     - 'direct' |
| `/print/cv` | http | user |  |  | `hr_skills` | `HrEmployeeCV.print_employee_cv` |  |
| `/my/timesheets` | http | user |  | True | `hr_timesheet` | `TimesheetCustomerPortal.portal_my_timesheets` |  |
| `/my/timesheets/page/<int:page>` | http | user |  | True | `hr_timesheet` | `TimesheetCustomerPortal.portal_my_timesheets` |  |
| `/html_editor/attachment/remove` | jsonrpc | user |  | True | `html_editor` | `HTML_Editor.remove` | Removes a web-based image attachment if it is used by no view (template)  Returns a dict mapping attachments which would not be removed (if any) mapped to the views preventing their removal |
| `/web_editor/get_image_info` | jsonrpc | user |  | True | `html_editor` | `HTML_Editor.get_image_info` | This route is used to determine the information of an attachment so that it can be used as a base to modify it again (crop/optimization/filters). |
| `/html_editor/get_image_info` | jsonrpc | user |  | True | `html_editor` | `HTML_Editor.get_image_info` | This route is used to determine the information of an attachment so that it can be used as a base to modify it again (crop/optimization/filters). |
| `/web_editor/video_url/data` | jsonrpc | user |  | True | `html_editor` | `HTML_Editor.video_url_data` |  |
| `/html_editor/video_url/data` | jsonrpc | user |  | True | `html_editor` | `HTML_Editor.video_url_data` |  |
| `/web_editor/attachment/add_data` | jsonrpc | user | ["POST"] | True | `html_editor` | `HTML_Editor.add_data` |  |
| `/html_editor/attachment/add_data` | jsonrpc | user | ["POST"] | True | `html_editor` | `HTML_Editor.add_data` |  |
| `/web_editor/attachment/add_url` | jsonrpc | user | ["POST"] | True | `html_editor` | `HTML_Editor.add_url` |  |
| `/html_editor/attachment/add_url` | jsonrpc | user | ["POST"] | True | `html_editor` | `HTML_Editor.add_url` |  |
| `/web_editor/modify_image/<model("ir.attachment"):attachment>` | jsonrpc | user |  | True | `html_editor` | `HTML_Editor.modify_image` | Creates a modified copy of an attachment and returns its image_src to be inserted into the DOM. |
| `/html_editor/modify_image/<model("ir.attachment"):attachment>` | jsonrpc | user |  | True | `html_editor` | `HTML_Editor.modify_image` | Creates a modified copy of an attachment and returns its image_src to be inserted into the DOM. |
| `/web_editor/save_library_media` | jsonrpc | user | ["POST"] |  | `html_editor` | `HTML_Editor.save_library_media` | Saves images from the media library as new attachments, making them dynamic SVGs if needed.     media = {         <media_id>: {             'query': 'space separated search terms',             'is_dynamic_svg': True/False,             'dynamic_colors': maps color names to their color,         }, ... |
| `/html_editor/save_library_media` | jsonrpc | user | ["POST"] |  | `html_editor` | `HTML_Editor.save_library_media` | Saves images from the media library as new attachments, making them dynamic SVGs if needed.     media = {         <media_id>: {             'query': 'space separated search terms',             'is_dynamic_svg': True/False,             'dynamic_colors': maps color names to their color,         }, ... |
| `/web_editor/shape/<module>/<path:filename>` | http | public |  | True | `html_editor` | `HTML_Editor.shape` | Returns a color-customized svg (background shape or illustration). |
| `/html_editor/shape/<module>/<path:filename>` | http | public |  | True | `html_editor` | `HTML_Editor.shape` | Returns a color-customized svg (background shape or illustration). |
| `/web_editor/image_shape/<string:img_key>/<module>/<path:filename>` | http | public |  | True | `html_editor` | `HTML_Editor.image_shape` |  |
| `/html_editor/image_shape/<string:img_key>/<module>/<path:filename>` | http | public |  | True | `html_editor` | `HTML_Editor.image_shape` |  |
| `/web_editor/generate_text` | jsonrpc | user |  |  | `html_editor` | `HTML_Editor.generate_text` |  |
| `/html_editor/generate_text` | jsonrpc | user |  |  | `html_editor` | `HTML_Editor.generate_text` |  |
| `/web_editor/get_ice_servers` | jsonrpc | user |  |  | `html_editor` | `HTML_Editor.get_ice_servers` |  |
| `/html_editor/get_ice_servers` | jsonrpc | user |  |  | `html_editor` | `HTML_Editor.get_ice_servers` |  |
| `/web_editor/bus_broadcast` | jsonrpc | user |  |  | `html_editor` | `HTML_Editor.bus_broadcast` |  |
| `/html_editor/bus_broadcast` | jsonrpc | user |  |  | `html_editor` | `HTML_Editor.bus_broadcast` |  |
| `/html_editor/link_preview_external` | jsonrpc | public | ["POST"] |  | `html_editor` | `HTML_Editor.link_preview_metadata` |  |
| `/html_editor/link_preview_internal` | jsonrpc | user | ["POST"] |  | `html_editor` | `HTML_Editor.link_preview_metadata_internal` |  |
| `/html_editor/media_library_search` | jsonrpc | user |  | True | `html_editor` | `HTML_Editor.media_library_search` |  |
| `/website/translations` | http | public |  |  | `http_routing` | `Routing.get_website_translations` |  |
| `/web/session/logout` | http | user |  | True | `http_routing` | `SessionWebsite.logout` |  |
| `/im_livechat/session/update_note` | jsonrpc | user | ["POST"] |  | `im_livechat` | `LivechatChannelController.livechat_session_update_note` | Internal users having the rights to read the session can update its note. |
| `/im_livechat/session/update_status` | jsonrpc | user | ["POST"] |  | `im_livechat` | `LivechatChannelController.livechat_session_update_status` | Internal users having the rights to read the session can update its status. |
| `/im_livechat/conversation/update_tags` | jsonrpc | user | ["POST"] |  | `im_livechat` | `LivechatChannelController.livechat_conversation_update_tags` | Add or remove tags from a live chat conversation. |
| `/im_livechat/conversation/write_expertises` | jsonrpc | user | ["POST"] |  | `im_livechat` | `LivechatChannelController.livechat_conversation_write_expertises` |  |
| `/im_livechat/conversation/create_and_link_expertise` | jsonrpc | user | ["POST"] |  | `im_livechat` | `LivechatChannelController.livechat_conversation_create_and_link_expertise` |  |
| `/chatbot/restart` | jsonrpc | public |  |  | `im_livechat` | `LivechatChatbotScriptController.chatbot_restart` |  |
| `/chatbot/answer/save` | jsonrpc | public |  |  | `im_livechat` | `LivechatChatbotScriptController.chatbot_save_answer` |  |
| `/chatbot/step/trigger` | jsonrpc | public |  |  | `im_livechat` | `LivechatChatbotScriptController.chatbot_trigger_step` |  |
| `/chatbot/step/validate_email` | jsonrpc | public |  |  | `im_livechat` | `LivechatChatbotScriptController.chatbot_validate_email` |  |
| `/im_livechat/external_lib.<any(css,js):ext>` | http | public |  |  | `im_livechat` | `LivechatController.external_lib` | Preserve compatibility with legacy livechat imports. Only serves javascript since the css will be fetched by the shadow DOM of the livechat to avoid conflicts. |
| `/im_livechat/assets_embed.<any(css, js):ext>` | http | public |  |  | `im_livechat` | `LivechatController.assets_embed` |  |
| `/im_livechat/font-awesome` | http | none |  |  | `im_livechat` | `LivechatController.fontawesome` |  |
| `/im_livechat/odoo_ui_icons` | http | none |  |  | `im_livechat` | `LivechatController.odoo_ui_icons` |  |
| `/im_livechat/emoji_bundle` | http | public |  |  | `im_livechat` | `LivechatController.get_emoji_bundle` |  |
| `/im_livechat/support/<int:channel_id>` | http | public |  |  | `im_livechat` | `LivechatController.support_page` |  |
| `/im_livechat/loader/<int:channel_id>` | http | public |  |  | `im_livechat` | `LivechatController.loader` |  |
| `/im_livechat/get_session` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatController.get_session` |  |
| `/im_livechat/feedback` | jsonrpc | public |  |  | `im_livechat` | `LivechatController.feedback` |  |
| `/im_livechat/history` | jsonrpc | public |  |  | `im_livechat` | `LivechatController.history_pages` |  |
| `/im_livechat/email_livechat_transcript` | jsonrpc | user |  |  | `im_livechat` | `LivechatController.email_livechat_transcript` |  |
| `/im_livechat/download_transcript/<int:channel_id>` | http | public |  |  | `im_livechat` | `LivechatController.download_livechat_transcript` |  |
| `/im_livechat/visitor_leave_session` | jsonrpc | public |  |  | `im_livechat` | `LivechatController.visitor_leave_session` | Called when the livechat visitor leaves the conversation. This will clean the chat request and warn the operator that the conversation is over. This allows also to re-send a new chat request to the visitor, as while the visitor is in conversation with an operator, it's not possible to send the visit |
| `/web/tests/livechat` | http | user |  |  | `im_livechat` | `WebClient.test_external_livechat` |  |
| `/im_livechat/cors/attachment/upload` | http | public |  |  | `im_livechat` | `LivechatAttachmentController.im_livechat_attachment_upload` |  |
| `/im_livechat/cors/attachment/delete` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatAttachmentController.im_livechat_attachment_delete` |  |
| `/im_livechat/cors/channel/messages` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatChannelController.livechat_channel_messages` |  |
| `/im_livechat/cors/channel/mark_as_read` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatChannelController.livechat_channel_mark_as_read` |  |
| `/im_livechat/cors/channel/notify_typing` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatChannelController.livechat_channel_notify_typing` |  |
| `/chatbot/cors/restart` | jsonrpc | public |  |  | `im_livechat` | `CorsLivechatChatbotScriptController.cors_chatbot_restart` |  |
| `/chatbot/cors/answer/save` | jsonrpc | public |  |  | `im_livechat` | `CorsLivechatChatbotScriptController.cors_chatbot_save_answer` |  |
| `/chatbot/cors/step/trigger` | jsonrpc | public |  |  | `im_livechat` | `CorsLivechatChatbotScriptController.cors_chatbot_trigger_step` |  |
| `/chatbot/cors/step/validate_email` | jsonrpc | public |  |  | `im_livechat` | `CorsLivechatChatbotScriptController.cors_chatbot_validate_email` |  |
| `/im_livechat/cors/link_preview` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatLinkPreviewController.livechat_link_preview` |  |
| `/im_livechat/cors/link_preview/hide` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatLinkPreviewController.livechat_link_preview_hide` |  |
| `/im_livechat/cors/visitor_leave_session` | jsonrpc | public |  |  | `im_livechat` | `CorsLivechatController.cors_visitor_leave_session` |  |
| `/im_livechat/cors/feedback` | jsonrpc | public |  |  | `im_livechat` | `CorsLivechatController.cors_feedback` |  |
| `/im_livechat/cors/history` | jsonrpc | public |  |  | `im_livechat` | `CorsLivechatController.cors_history_pages` |  |
| `/im_livechat/cors/download_transcript/<int:channel_id>` | http | public |  |  | `im_livechat` | `CorsLivechatController.cors_download_livechat_transcript` |  |
| `/im_livechat/cors/get_session` | jsonrpc | public | ["POST"] |  | `im_livechat` | `CorsLivechatController.cors_get_session` |  |
| `/im_livechat/cors/init` | jsonrpc | public |  |  | `im_livechat` | `CorsLivechatController.cors_livechat_init` |  |
| `/im_livechat/cors/message/reaction` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatMessageReactionController.livechat_message_reaction` |  |
| `/im_livechat/cors/rtc/channel/join_call` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatRtcController.livechat_channel_call_join` |  |
| `/im_livechat/cors/rtc/channel/leave_call` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatRtcController.livechat_channel_call_leave` |  |
| `/im_livechat/cors/rtc/session/update_and_broadcast` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatRtcController.livechat_session_update_and_broadcast` |  |
| `/im_livechat/cors/rtc/session/notify_call_members` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatRtcController.livechat_session_call_notify` |  |
| `/im_livechat/cors/channel/ping` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatRtcController.livechat_channel_ping` |  |
| `/im_livechat/cors/message/post` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatThreadController.livechat_message_post` |  |
| `/im_livechat/cors/message/update_content` | jsonrpc | public | ["POST"] |  | `im_livechat` | `LivechatThreadController.livechat_message_update_content` |  |
| `/im_livechat/cors/action` | jsonrpc | public | ["POST"] |  | `im_livechat` | `WebClient.livechat_action` |  |
| `/im_livechat/cors/data` | jsonrpc | public | ["POST"] |  | `im_livechat` | `WebClient.livechat_data` |  |
| `/hw_proxy/scale_read` | jsonrpc | none |  |  | `iot_drivers` | `ScaleReadHardwareProxy.scale_read` |  |
| `/nemhandel/webhook/new-message` | http | public | ["POST"] |  | `l10n_dk_nemhandel` | `NemhandelWebhookController.webhook_nemhandel_new_message` |  |
| `/nemhandel/webhook/message-state-update` | http | public | ["POST"] |  | `l10n_dk_nemhandel` | `NemhandelWebhookController.webhook_nemhandel_message_update` |  |
| `/nemhandel/webhook/user-state-update` | http | public | ["POST"] |  | `l10n_dk_nemhandel` | `NemhandelWebhookController.webhook_nemhandel_user_update` |  |
| `/api/signaturit_authentication_status/1/webhooks` | http | public | ["POST"] |  | `l10n_fr_pdp` | `IapAuthenticationWebhook.notify_authentication_status` |  |
| `/peppol/webhook/new-regulatory-message` | http | public | ["POST"] |  | `l10n_fr_pdp` | `PdpWebhookController.webhook_regulatory_message` |  |
| `/l10n_id_efaktur_coretax/download_attachments/<models("ir.attachment"):attachments>` | http | user |  |  | `l10n_id_efaktur_coretax` | `EfakturDownloadController.download_invoice_attachments` |  |
| `/portal/state_infos/<model("res.country.state"):state>` | jsonrpc | public | ["POST"] | True | `l10n_pe` | `L10nPEPortalAccount.state_infos` |  |
| `/portal/city_infos/<model("res.city"):city>` | jsonrpc | public | ["POST"] | True | `l10n_pe` | `L10nPEPortalAccount.city_infos` |  |
| `/l10n_ro_edi/authorize/<int:company_id>` | http | user |  |  | `l10n_ro_edi` | `L10nRoEdiController.authorize` | Generate Authorization Token to acquire access_key for requesting Access Token |
| `/l10n_ro_edi/callback/<int:company_id>` | http | user |  |  | `l10n_ro_edi` | `L10nRoEdiController.callback` | Use the acquired access_key to request access & refresh token from ANAF |
| `/invoice/ecpay/agreed_invoice_allowance/<int:invoice_id>` | http | public | ["POST"] |  | `l10n_tw_edi_ecpay` | `EcpayInvoiceController.agreed_invoice_allowance` |  |
| `/shop/l10n_tw_invoicing_info` | http | public | ["GET"] | True | `l10n_tw_edi_ecpay_website_sale` | `WebsiteSaleL10nTW.l10n_tw_invoicing_info_get` |  |
| `/shop/l10n_tw_invoicing_info/submit` | http | public | ["POST"] | True | `l10n_tw_edi_ecpay_website_sale` | `WebsiteSaleL10nTW.l10n_tw_invoicing_info_post` |  |
| `/payment/ecpay/check_mobile_barcode/<int:sale_order_id>` | jsonrpc | public |  |  | `l10n_tw_edi_ecpay_website_sale` | `WebsiteSaleL10nTW.check_mobile_barcode` |  |
| `/payment/ecpay/check_love_code/<int:sale_order_id>` | jsonrpc | public |  |  | `l10n_tw_edi_ecpay_website_sale` | `WebsiteSaleL10nTW.check_love_code` |  |
| `/r/<string:code>` | http | public |  | True | `link_tracker` | `LinkTracker.full_url_redirect` |  |
| `/my/loyalty_card/<int:card_id>/history` | http | user |  | True | `loyalty` | `CustomerPortalLoyalty.portal_my_loyalty_card_history` |  |
| `/my/loyalty_card/<int:card_id>/history/page/<int:page>` | http | user |  | True | `loyalty` | `CustomerPortalLoyalty.portal_my_loyalty_card_history` |  |
| `/my/loyalty_card/<int:card_id>/values` | jsonrpc | user |  |  | `loyalty` | `CustomerPortalLoyalty.portal_get_card_history_values` | Retrieve card history values for portal card dialog.  :param card_id(str): The ID of the loyalty card. :return(dict): A dictionary with card history values. |
| `/lunch/infos` | jsonrpc | user |  |  | `lunch` | `LunchController.infos` |  |
| `/lunch/trash` | jsonrpc | user |  |  | `lunch` | `LunchController.trash` |  |
| `/lunch/pay` | jsonrpc | user |  |  | `lunch` | `LunchController.pay` |  |
| `/lunch/payment_message` | jsonrpc | user |  |  | `lunch` | `LunchController.payment_message` |  |
| `/lunch/user_location_set` | jsonrpc | user |  |  | `lunch` | `LunchController.set_user_location` |  |
| `/lunch/user_location_get` | jsonrpc | user |  |  | `lunch` | `LunchController.get_user_location` |  |
| `/mail/attachment/upload` | http | public | ["POST"] |  | `mail` | `AttachmentController.mail_attachment_upload` |  |
| `/mail/attachment/delete` | jsonrpc | public | ["POST"] |  | `mail` | `AttachmentController.mail_attachment_delete` |  |
| `/mail/attachment/zip` | http | public | ["POST"] |  | `mail` | `AttachmentController.mail_attachment_get_zip` | route to get the zip file of the attachments. :param file_ids: ids of the files to zip. :param zip_name: name of the zip file. |
| `/mail/attachment/pdf_first_page/<int:attachment_id>` | http | public | ["GET"] |  | `mail` | `AttachmentController.mail_attachment_pdf_first_page` | Returns the first page of a pdf. |
| `/mail/attachment/update_thumbnail` | jsonrpc | public | ["POST"] |  | `mail` | `AttachmentController.mail_attachement_update_thumbnail` | Updates the thumbnail of an attachment. |
| `/mail/message/translate` | jsonrpc | user |  |  | `mail` | `GoogleTranslateController.translate` |  |
| `/mail/guest/update_name` | jsonrpc | public | ["POST"] |  | `mail` | `GuestController.mail_guest_update_name` |  |
| `/mail/set_manual_im_status` | jsonrpc | user | ["POST"] |  | `mail` | `ImStatusController.set_manual_im_status` |  |
| `/mail/link_preview` | jsonrpc | public | ["POST"] |  | `mail` | `LinkPreviewController.mail_link_preview` |  |
| `/mail/link_preview/hide` | jsonrpc | public | ["POST"] |  | `mail` | `LinkPreviewController.mail_link_preview_hide` |  |
| `/mail/view` | http | public |  |  | `mail` | `MailController.mail_action_view` | Generic access point from notification emails. The heuristic to    choose where to redirect the user is the following :  - find a public URL - if none found  - users with a read access are redirected to the document  - users without read access are redirected to the Messaging  - not logged users are |
| `/mail/unfollow` | http | public |  |  | `mail` | `MailController.mail_action_unfollow` |  |
| `/mail/message/<int:message_id>` | http | public |  |  | `mail` | `MailController.mail_thread_message_redirect` |  |
| `/web_editor/font_to_img/<icon>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/web_editor/font_to_img/<icon>/<color>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/web_editor/font_to_img/<icon>/<color>/<int:size>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/web_editor/font_to_img/<icon>/<color>/<int:width>x<int:height>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/web_editor/font_to_img/<icon>/<color>/<int:size>/<int:alpha>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/web_editor/font_to_img/<icon>/<color>/<int:width>x<int:height>/<int:alpha>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/web_editor/font_to_img/<icon>/<color>/<bg>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/web_editor/font_to_img/<icon>/<color>/<bg>/<int:size>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/web_editor/font_to_img/<icon>/<color>/<bg>/<int:width>x<int:height>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/web_editor/font_to_img/<icon>/<color>/<bg>/<int:width>x<int:height>/<int:alpha>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>/<color>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>/<color>/<int:size>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>/<color>/<int:width>x<int:height>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>/<color>/<int:size>/<int:alpha>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>/<color>/<int:width>x<int:height>/<int:alpha>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>/<color>/<bg>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>/<color>/<bg>/<int:size>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>/<color>/<bg>/<int:width>x<int:height>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/font_to_img/<icon>/<color>/<bg>/<int:width>x<int:height>/<int:alpha>` | http | none |  |  | `mail` | `MailController.export_icon_to_png` | This method converts an unicode character to an image (using Font Awesome font by default) and is used only for mass mailing because custom fonts are not supported in mail. :param icon : decimal encoding of unicode character :param color : RGB code of the color :param bg : RGB code of the background |
| `/mail/inbox/messages` | jsonrpc | user | ["POST"] |  | `mail` | `MailboxController.discuss_inbox_messages` |  |
| `/mail/history/messages` | jsonrpc | user | ["POST"] |  | `mail` | `MailboxController.discuss_history_messages` |  |
| `/mail/starred/messages` | jsonrpc | user | ["POST"] |  | `mail` | `MailboxController.discuss_starred_messages` |  |
| `/mail/message/reaction` | jsonrpc | public | ["POST"] |  | `mail` | `MessageReactionController.mail_message_reaction` |  |
| `/mail/thread/messages` | jsonrpc | user | ["POST"] |  | `mail` | `ThreadController.mail_thread_messages` |  |
| `/mail/thread/recipients` | jsonrpc | user | ["POST"] |  | `mail` | `ThreadController.mail_thread_recipients` | Fetch discussion-based suggested recipients, creating partners on the fly |
| `/mail/thread/recipients/fields` | jsonrpc | user | ["POST"] |  | `mail` | `ThreadController.mail_thread_recipients_fields` |  |
| `/mail/thread/recipients/get_suggested_recipients` | jsonrpc | user | ["POST"] |  | `mail` | `ThreadController.mail_thread_recipients_get_suggested_recipients` | This method returns the suggested recipients with updates coming from the frontend. :param thread_model: Model on which we are currently working on. :param thread_id: ID of the document we need to compute :param partner_ids: IDs of new customers that were edited on the frontend, usually only the cus |
| `/mail/partner/from_email` | jsonrpc | user | ["POST"] |  | `mail` | `ThreadController.mail_thread_partner_from_email` |  |
| `/mail/read_subscription_data` | jsonrpc | user | ["POST"] |  | `mail` | `ThreadController.read_subscription_data` | Computes: - message_subtype_data: data about document subtypes: which are     available, which are followed if any |
| `/mail/message/post` | jsonrpc | public | ["POST"] |  | `mail` | `ThreadController.mail_message_post` |  |
| `/mail/message/update_content` | jsonrpc | public | ["POST"] |  | `mail` | `ThreadController.mail_message_update_content` |  |
| `/mail/thread/unsubscribe` | jsonrpc | user | ["POST"] |  | `mail` | `ThreadController.mail_thread_unsubscribe` |  |
| `/mail/thread/subscribe` | jsonrpc | user | ["POST"] |  | `mail` | `ThreadController.mail_thread_subscribe` |  |
| `/mail/action` | jsonrpc | public | ["POST"] |  | `mail` | `WebclientController.mail_action` | Execute actions and returns data depending on request parameters. This is similar to /mail/data except this method can have side effects. |
| `/mail/data` | jsonrpc | public | ["POST"] |  | `mail` | `WebclientController.mail_data` | Returns data depending on request parameters. This is similar to /mail/action except this method should be read-only. |
| `/websocket/update_bus_presence` | jsonrpc | public |  |  | `mail` | `WebsocketControllerPresence.update_bus_presence` | Manually update presence of current user, useful when implementing custom websocket code. This is mainly used by the system.sh. |
| `/discuss/channel/members` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.discuss_channel_members` |  |
| `/discuss/channel/update_avatar` | jsonrpc | user | ["POST"] |  | `mail` | `ChannelController.discuss_channel_avatar_update` |  |
| `/discuss/channel/messages` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.discuss_channel_messages` |  |
| `/discuss/channel/pinned_messages` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.discuss_channel_pins` |  |
| `/discuss/channel/mark_as_read` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.discuss_channel_mark_as_read` |  |
| `/discuss/channel/set_new_message_separator` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.discuss_channel_set_new_message_separator` |  |
| `/discuss/channel/notify_typing` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.discuss_channel_notify_typing` |  |
| `/discuss/channel/attachments` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.load_attachments` | Load attachments of a channel. If before is set, load attachments older than the given id. :param channel_id: id of the channel :param limit: maximum number of attachments to return :param before: id of the attachment from which to load older attachments |
| `/discuss/channel/join` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.discuss_channel_join` |  |
| `/discuss/channel/sub_channel/create` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.discuss_channel_sub_channel_create` |  |
| `/discuss/channel/sub_channel/fetch` | jsonrpc | public | ["POST"] |  | `mail` | `ChannelController.discuss_channel_sub_channel_fetch` |  |
| `/discuss/channel/sub_channel/delete` | jsonrpc | user | ["POST"] |  | `mail` | `ChannelController.discuss_delete_sub_channel` |  |
| `/discuss/gif/search` | jsonrpc | user |  |  | `mail` | `DiscussGifController.search` |  |
| `/discuss/gif/categories` | jsonrpc | user |  |  | `mail` | `DiscussGifController.categories` |  |
| `/discuss/gif/add_favorite` | jsonrpc | user |  |  | `mail` | `DiscussGifController.add_favorite` |  |
| `/discuss/gif/favorites` | jsonrpc | user |  |  | `mail` | `DiscussGifController.get_favorites` |  |
| `/discuss/gif/remove_favorite` | jsonrpc | user |  |  | `mail` | `DiscussGifController.remove_favorite` |  |
| `/chat/<string:create_token>` | http | public | ["GET"] |  | `mail` | `PublicPageController.discuss_channel_chat_from_token` |  |
| `/chat/<string:create_token>/<string:channel_name>` | http | public | ["GET"] |  | `mail` | `PublicPageController.discuss_channel_chat_from_token` |  |
| `/meet/<string:create_token>` | http | public | ["GET"] |  | `mail` | `PublicPageController.discuss_channel_meet_from_token` |  |
| `/meet/<string:create_token>/<string:channel_name>` | http | public | ["GET"] |  | `mail` | `PublicPageController.discuss_channel_meet_from_token` |  |
| `/chat/<int:channel_id>/<string:invitation_token>` | http | public | ["GET"] |  | `mail` | `PublicPageController.discuss_channel_invitation` |  |
| `/discuss/channel/<int:channel_id>` | http | public | ["GET"] |  | `mail` | `PublicPageController.discuss_channel` |  |
| `/mail/rtc/session/notify_call_members` | jsonrpc | public | ["POST"] |  | `mail` | `RtcController.session_call_notify` | Sends content to other session of the same channel, only works if the user is the user of that session. This is used to send peer to peer information between sessions.  :param peer_notifications: list of tuple with the following elements:     - int sender_session_id: id of the session from which the |
| `/mail/rtc/session/update_and_broadcast` | jsonrpc | public | ["POST"] |  | `mail` | `RtcController.session_update_and_broadcast` | Update a RTC session and broadcasts the changes to the members of its channel, only works of the user is the user of that session. :param int session_id: id of the session to update :param dict values: write dict for the fields to update |
| `/mail/rtc/channel/join_call` | jsonrpc | public | ["POST"] |  | `mail` | `RtcController.channel_call_join` | Joins the RTC call of a channel if the user is a member of that channel :param int channel_id: id of the channel to join |
| `/mail/rtc/channel/leave_call` | jsonrpc | public | ["POST"] |  | `mail` | `RtcController.channel_call_leave` | Disconnects the current user from a rtc call and clears any invitation sent to that user on this channel :param int channel_id: id of the channel from which to disconnect :param int session_id: id of the leaving session |
| `/mail/rtc/channel/upgrade_connection` | jsonrpc | user | ["POST"] |  | `mail` | `RtcController.channel_upgrade` |  |
| `/mail/rtc/channel/cancel_call_invitation` | jsonrpc | public | ["POST"] |  | `mail` | `RtcController.channel_call_cancel_invitation` | :param member_ids: members whose invitation is to cancel :type member_ids: list(int) or None |
| `/mail/rtc/audio_worklet_processor_v2` | http | public | ["GET"] |  | `mail` | `RtcController.audio_worklet_processor` | Returns a JS file that declares a WorkletProcessor class in a WorkletGlobalScope, which means that it cannot be added to the bundles like other assets. |
| `/discuss/channel/ping` | jsonrpc | public | ["POST"] |  | `mail` | `RtcController.channel_ping` |  |
| `/discuss/search` | jsonrpc | public | ["POST"] |  | `mail` | `SearchController.search` |  |
| `/discuss/settings/mute` | jsonrpc | user | ["POST"] |  | `mail` | `DiscussSettingsController.discuss_mute` | Mute notifications for the given number of minutes. :param minutes: (integer) number of minutes to mute notifications, -1 means mute until the user unmutes :param channel_id: (integer) id of the discuss.channel record |
| `/discuss/settings/custom_notifications` | jsonrpc | user | ["POST"] |  | `mail` | `DiscussSettingsController.discuss_custom_notifications` | Set custom notifications for the given channel or general user settings. :param custom_notifications: (false\|all\|mentions\|no_notif) custom notifications to set :param channel_id: (integer) id of the discuss.channel record, if not set, set for res.users.settings |
| `/discuss/voice/worklet_processor` | http | public | ["GET"] |  | `mail` | `VoiceController.voice_worklet_processor` |  |
| `/groups` | http | public |  | True | `mail_group` | `PortalMailGroup.groups_index` | View of the group lists. Allow the users to subscribe and unsubscribe. |
| `/groups/<model("mail.group"):group>` | http | public |  | True | `mail_group` | `PortalMailGroup.group_view_messages` |  |
| `/groups/<model("mail.group"):group>/page/<int:page>` | http | public |  | True | `mail_group` | `PortalMailGroup.group_view_messages` |  |
| `/groups/<model("mail.group"):group>/<model("mail.group.message"):message>` | http | public |  | True | `mail_group` | `PortalMailGroup.group_view_message` |  |
| `/groups/<model("mail.group"):group>/<model("mail.group.message"):message>/get_replies` | jsonrpc | public | ["POST"] | True | `mail_group` | `PortalMailGroup.group_message_get_replies` |  |
| `/group/<int:group_id>/unsubscribe_oneclick` | http | public | ["POST"] | True | `mail_group` | `PortalMailGroup.group_unsubscribe_oneclick` | Unsubscribe a given user from a given group. One-click unsubscribe allow mail user agent to propose a one click button to the user to unsubscribe as defined in rfc8058. Only POST method is allowed preventing the risk that anti-spam trigger unwanted unsubscribe (scenario explained in the same rfc).   |
| `/group/subscribe` | jsonrpc | public |  | True | `mail_group` | `PortalMailGroup.group_subscribe` | Subscribe the current logged user or the given email address to the mailing list.  If the user is logged, the action is automatically done.  But if the user is not logged (public user) an email will be send with a token to confirm the action.  :param group_id: Id of the group :param email: Email to  |
| `/group/unsubscribe` | jsonrpc | public |  | True | `mail_group` | `PortalMailGroup.group_unsubscribe` | Unsubscribe the current logged user or the given email address to the mailing list.  If the user is logged, the action is automatically done.  But if the user is not logged (public user) an email will be send with a token to confirm the action.  :param group_id: Id of the group :param email: Email t |
| `/group/subscribe-confirm` | http | public |  | True | `mail_group` | `PortalMailGroup.group_subscribe_confirm` | Confirm the subscribe / unsubscribe action which was sent by email. |
| `/group/unsubscribe-confirm` | http | public |  | True | `mail_group` | `PortalMailGroup.group_unsubscribe_confirm` | Confirm the subscribe / unsubscribe action which was sent by email. |
| `/mail_client_extension/auth` | http | user | ["GET"] | True | `mail_plugin` | `Authenticate.auth` | Once authenticated this route renders the view that shows an app wants to access the system. The user is invited to allow or deny the app. The form posts to `/mail_client_extension/auth/confirm`.  old route name "/mail_client_extension/auth is deprecated as of saas-14.3,it is not needed for newer ve |
| `/mail_plugin/auth` | http | user | ["GET"] | True | `mail_plugin` | `Authenticate.auth` | Once authenticated this route renders the view that shows an app wants to access the system. The user is invited to allow or deny the app. The form posts to `/mail_client_extension/auth/confirm`.  old route name "/mail_client_extension/auth is deprecated as of saas-14.3,it is not needed for newer ve |
| `/mail_client_extension/auth/confirm` | http | user | ["POST"] |  | `mail_plugin` | `Authenticate.auth_confirm` | Called by the `app_auth` template. If the user decided to allow the app to access the system, a temporary auth code is generated and they are redirected to `redirect` with this code in the URL. It should redirect to the app, and the app should then exchange this auth code for an access token by call |
| `/mail_plugin/auth/confirm` | http | user | ["POST"] |  | `mail_plugin` | `Authenticate.auth_confirm` | Called by the `app_auth` template. If the user decided to allow the app to access the system, a temporary auth code is generated and they are redirected to `redirect` with this code in the URL. It should redirect to the app, and the app should then exchange this auth code for an access token by call |
| `/mail_plugin/auth/check_version` | jsonrpc | none | ["POST", "OPTIONS"] |  | `mail_plugin` | `Authenticate.auth_check_version` | Allow to know if the module is installed and which addin version is supported. |
| `/mail_client_extension/auth/access_token` | jsonrpc | none | ["POST", "OPTIONS"] |  | `mail_plugin` | `Authenticate.auth_access_token` | Called by the external app to exchange an auth code, which is temporary and was passed in a URL, for an access token, which is permanent, and can be used in the `Authorization` header to authorize subsequent requests  old route name "/mail_client_extension/auth/access_token is deprecated as of saas- |
| `/mail_plugin/auth/access_token` | jsonrpc | none | ["POST", "OPTIONS"] |  | `mail_plugin` | `Authenticate.auth_access_token` | Called by the external app to exchange an auth code, which is temporary and was passed in a URL, for an access token, which is permanent, and can be used in the `Authorization` header to authorize subsequent requests  old route name "/mail_client_extension/auth/access_token is deprecated as of saas- |
| `/mail_client_extension/modules/get` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.modules_get` | deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary for supporting older versions |
| `/mail_plugin/partner/enrich_and_create_company` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.res_partner_enrich_and_create_company` | Route used when the user clicks on the create and enrich partner button it will try to find a company using IAP, if a company is found the enriched company will then be created in the database |
| `/mail_plugin/partner/enrich_and_update_company` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.res_partner_enrich_and_update_company` | Enriches an existing company using IAP |
| `/mail_client_extension/partner/get` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.res_partner_get` | returns a partner given it's id or an email and a name. In case the partner does not exist, we return partner having an id -1, we also look if an existing company matching the contact exists in the database, if none is found a new company is enriched and created automatically  old route name "/mail_ |
| `/mail_plugin/partner/get` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.res_partner_get` | returns a partner given it's id or an email and a name. In case the partner does not exist, we return partner having an id -1, we also look if an existing company matching the contact exists in the database, if none is found a new company is enriched and created automatically  old route name "/mail_ |
| `/mail_plugin/partner/search` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.res_partners_search` | Used for the plugin search contact functionality where the user types a string query in order to search for matching contacts, the string query can either be the name of the contact, it's reference or it's email. We choose these fields because these are probably the most interesting fields that the  |
| `/mail_client_extension/partner/create` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.res_partner_create` | params email: email of the new partner params name: name of the new partner params company: parent company id of the new partner |
| `/mail_plugin/partner/create` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.res_partner_create` | params email: email of the new partner params name: name of the new partner params company: parent company id of the new partner |
| `/mail_plugin/log_mail_content` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.log_mail_content` | Log the email on the given record.  :param model: Model of the record on which we want to log the email :param res_id: ID of the record :param message: Body of the email :param attachments: List of attachments of the email.     List of tuple: (filename, base 64 encoded content) |
| `/mail_plugin/get_translations` | jsonrpc | outlook |  |  | `mail_plugin` | `MailPluginController.get_translations` |  |
| `/cards/<string:card_slug>/card.jpg` | http | public |  | True | `marketing_card` | `MarketingCardController.card_campaign_image` |  |
| `/cards/<int:card_id>/card.jpg` | http | public |  | True | `marketing_card` | `MarketingCardController.card_campaign_image` |  |
| `/cards/<string:card_slug>/preview` | http | public |  | True | `marketing_card` | `MarketingCardController.card_campaign_preview` | Route for users to preview their card and share it on their social platforms. |
| `/cards/<int:card_id>/preview` | http | public |  | True | `marketing_card` | `MarketingCardController.card_campaign_preview` | Route for users to preview their card and share it on their social platforms. |
| `/cards/<string:card_slug>/redirect` | http | public |  | True | `marketing_card` | `MarketingCardController.card_campaign_redirect` | Route to redirect users to the target url, or display the opengraph embed text for web crawlers.  When a user posts a link on an application supporting opengraph, the application will follow the link to fetch specific meta tags on the web page to get preview information such as a preview card. The " |
| `/cards/<int:card_id>/redirect` | http | public |  | True | `marketing_card` | `MarketingCardController.card_campaign_redirect` | Route to redirect users to the target url, or display the opengraph embed text for web crawlers.  When a user posts a link on an application supporting opengraph, the application will follow the link to fetch specific meta tags on the web page to get preview information such as a preview card. The " |
| `/mail/mailing/<int:mailing_id>/unsubscribe` | http | public |  | True | `mass_mailing` | `MailingLegacy.mailing_unsubscribe` | Old route, using mail/mailing prefix, and outdated parameter names |
| `/mailing/my` | http | user |  | True | `mass_mailing` | `MassMailController.mailing_my` |  |
| `/mailing/<int:mailing_id>/unsubscribe_oneclick` | http | public | ["POST"] | True | `mass_mailing` | `MassMailController.mailing_unsubscribe_oneclick` |  |
| `/mailing/<int:mailing_id>/confirm_unsubscribe` | http | public |  | True | `mass_mailing` | `MassMailController.mailing_confirm_unsubscribe` |  |
| `/mailing/confirm_unsubscribe` | http | public | ["POST"] | True | `mass_mailing` | `MassMailController.mailing_confirm_unsubscribe_post` |  |
| `/mailing/<int:mailing_id>/unsubscribe` | http | public |  | True | `mass_mailing` | `MassMailController.mailing_unsubscribe` |  |
| `/mailing/list/update` | jsonrpc | public |  |  | `mass_mailing` | `MassMailController.mailing_update_list_subscription` |  |
| `/mailing/feedback` | jsonrpc | public |  |  | `mass_mailing` | `MassMailController.mailing_send_feedback` | Feedback can be given after some actions, notably after opt-outing from mailing lists or adding an email in the blocklist.  This controller tries to write the customer feedback in the most relevant record. Feedback consists in two parts, the opt-out reason (based on data in 'mailing.subscription.opt |
| `/unsubscribe_from_list` | http | public |  | True | `mass_mailing` | `MassMailController.mailing_unsubscribe_placeholder_link` | Dummy route so placeholder is not prefixed by language, MUST have multilang=False |
| `/view` | http | user |  | True | `mass_mailing` | `MassMailController.mailing_view_in_browser_placeholder_link` | Route used to give an example of what would be when the user follows the placeholder links in the mailing editor. |
| `/mail/track/<int:mail_id>/<string:token>/blank.gif` | http | public |  |  | `mass_mailing` | `MassMailController.track_mail_open` | Email tracking. |
| `/r/<string:code>/m/<int:mailing_trace_id>` | http | public |  |  | `mass_mailing` | `MassMailController.full_url_redirect` |  |
| `/mailing/report/unsubscribe` | http | public |  | True | `mass_mailing` | `MassMailController.mailing_report_deactivate` |  |
| `/mailing/<int:mailing_id>/view` | http | public |  | True | `mass_mailing` | `MassMailController.mailing_view_in_browser` |  |
| `/mailing/blocklist/add` | jsonrpc | public |  |  | `mass_mailing` | `MassMailController.mail_blocklist_add` |  |
| `/mailing/blocklist/remove` | jsonrpc | public |  |  | `mass_mailing` | `MassMailController.mail_blocklist_remove` |  |
| `/mailing/mobile/preview` | http | user | ["GET"] | True | `mass_mailing` | `MassMailController.mass_mailing_preview_mobile_content` |  |
| `/sms/<int:mailing_id>/<string:trace_code>` | http | public |  | True | `mass_mailing_sms` | `MailingSMSController.blacklist_page` | Main entry point for unsubscribe links. Verify trace code (should match mailing and trace), then check number can be sanitized. |
| `/sms/<int:mailing_id>/unsubscribe/<string:trace_code>` | http | public |  | True | `mass_mailing_sms` | `MailingSMSController.blacklist_number` | Effectively opt-out or enter number in block list. |
| `/r/<string:code>/s/<int:sms_id_int>` | http | public |  |  | `mass_mailing_sms` | `MailingSMSController.sms_short_link_redirect` |  |
| `/microsoft_account/authentication` | http | public |  |  | `microsoft_account` | `MicrosoftAuth.oauth2callback` | This route/function is called by Microsoft when user Accept/Refuse the consent of Microsoft |
| `/microsoft_calendar/sync_data` | jsonrpc | user |  |  | `microsoft_calendar` | `MicrosoftCalendarController.microsoft_calendar_sync_data` | This route/function is called when we want to synchronize the system calendar with Microsoft Calendar. Function return a dictionary with the status :  need_config_from_admin, need_auth, need_refresh, sync_stopped, success if not calendar_event The dictionary may contains an url, to allow the system  |
| `/microsoft_outlook/confirm` | http | user |  |  | `microsoft_outlook` | `MicrosoftOutlookController.microsoft_outlook_callback` | Callback URL during the OAuth process.  Outlook redirects the user browser to this endpoint with the authorization code. We will fetch the refresh token and the access token thanks to this authorization code and save those values on the given mail server. |
| `/microsoft_outlook/iap_confirm` | http | user |  |  | `microsoft_outlook` | `MicrosoftOutlookController.microsoft_outlook_iap_callback` | Receive back the refresh token and access token from IAP.  The authentication process with IAP is done in 4 steps; 1. User database make a request to `<IAP>/api/mail_oauth/1/outlook` 2. User browser is redirected to the URL we received from IAP 3. User browser is redirected to `<IAP>/api/mail_oauth/ |
| `/my/productions` | http | user |  | True | `mrp_subcontracting` | `CustomerPortal.portal_my_productions` |  |
| `/my/productions/page/<int:page>` | http | user |  | True | `mrp_subcontracting` | `CustomerPortal.portal_my_productions` |  |
| `/my/productions/<int:picking_id>` | http | user | ["GET"] | True | `mrp_subcontracting` | `CustomerPortal.portal_my_production` |  |
| `/my/productions/<int:picking_id>/subcontracting_portal` | http | user | ["GET"] |  | `mrp_subcontracting` | `CustomerPortal.render_production_backend_view` |  |
| `/payment/pay` | http | public | ["GET"] | True | `payment` | `PaymentPortal.payment_pay` | Display the payment form with optional filtering of payment options.  The filtering takes place on the basis of provided parameters, if any. If a parameter is incorrect or malformed, it is skipped to avoid preventing the user from making the payment.  In addition to the desired filtering, a second o |
| `/my/payment_method` | http | user | ["GET"] | True | `payment` | `PaymentPortal.payment_method` | Display the form to manage payment methods.  :param dict kwargs: Optional data. This parameter is not used here :return: The rendered manage form :rtype: str |
| `/payment/transaction` | jsonrpc | public |  |  | `payment` | `PaymentPortal.payment_transaction` | Create a draft transaction and return its processing values.  :param float\|None amount: The amount to pay in the given currency.                           None if in a payment method validation operation :param int\|None currency_id: The currency of the transaction, as a `res.currency` id.          |
| `/payment/confirmation` | http | public | ["GET"] | True | `payment` | `PaymentPortal.payment_confirm` | Display the payment confirmation page to the user.  :param str tx_id: The transaction to confirm, as a `payment.transaction` id :param str access_token: The access token used to verify the user :param dict kwargs: Optional data. This parameter is not used here :raise NotFound: If the access token is |
| `/payment/archive_token` | jsonrpc | user |  |  | `payment` | `PaymentPortal.archive_token` | Check that a user has write access on a token and archive the token if so.  :param int token_id: The token to archive, as a `payment.token` id :return: None |
| `/payment/status` | http | public |  | True | `payment` | `PaymentPostProcessing.display_status` | Fetch the transaction and display it on the payment status page.  :param dict kwargs: Optional data. This parameter is not used here :return: The rendered status page :rtype: str |
| `/payment/status/poll` | jsonrpc | public |  |  | `payment` | `PaymentPostProcessing.poll_status` | Fetch the transaction and trigger its post-processing.  :return: The post-processing values of the transaction. :rtype: dict |
| `/payment/adyen/payment_methods` | jsonrpc | public |  |  | `payment_adyen` | `AdyenController.adyen_payment_methods` | Query the available payment methods based on the payment context.  :param int provider_id: The provider handling the transaction, as a `payment.provider` id :param dict formatted_amount: The Adyen-formatted amount. :param int partner_id: The partner making the transaction, as a `res.partner` id :ret |
| `/payment/adyen/payments` | jsonrpc | public |  |  | `payment_adyen` | `AdyenController.adyen_payments` | Make a payment request and process the payment data.  :param int provider_id: The provider handling the transaction, as a `payment.provider` id :param str reference: The reference of the transaction :param int converted_amount: The amount of the transaction in minor units of the currency :param int  |
| `/payment/adyen/payments/details` | jsonrpc | public |  |  | `payment_adyen` | `AdyenController.adyen_payment_details` | Submit the details of the additional actions and process the payment data.   The additional actions can have been performed both from the inline form or during a  redirection.  :param int provider_id: The provider handling the transaction, as a `payment.provider` id :param str reference: The referen |
| `/payment/adyen/return` | http | public |  |  | `payment_adyen` | `AdyenController.adyen_return_from_3ds_auth` | Process the authentication data sent by Adyen after redirection from the 3DS1 page.  The route is flagged with `save_session=False` to prevent the system from assigning a new session to the user if they are redirected to this route with a POST request. Indeed, as the session cookie is created withou |
| `expression` | http | public | ["POST"] |  | `payment_adyen` | `AdyenController.adyen_webhook` | Process the data sent by Adyen to the webhook based on the event code.  See https://docs.adyen.com/development-resources/webhooks/understand-notifications for the exhaustive list of event codes.  :return: The '[accepted]' string to acknowledge the notification :rtype: str |
| `expression` | http | public | ["POST"] |  | `payment_aps` | `APSController.aps_return_from_checkout` | Process the payment data sent by APS after redirection.  The route is flagged with `save_session=False` to prevent the system from assigning a new session to the user if they are redirected to this route with a POST request. Indeed, as the session cookie is created without a `SameSite` attribute, so |
| `expression` | http | public | ["POST"] |  | `payment_aps` | `APSController.aps_webhook` | Process the payment data sent by APS to the webhook.  See https://paymentservices-reference.payfort.com/docs/api/build/index.html#transaction-feedback.  :param dict data: The payment data. :return: The 'SUCCESS' string to acknowledge the notification :rtype: str |
| `expression` | http | public | ["GET"] |  | `payment_asiapay` | `AsiaPayController.asiapay_return_from_checkout` | Process the payment data sent by AsiaPay after redirection.  :param dict data: The payment data. |
| `expression` | http | public | ["POST"] |  | `payment_asiapay` | `AsiaPayController.asiapay_webhook` | Process the payment data sent by AsiaPay to the webhook.  :param dict data: The payment data. :return: The 'OK' string to acknowledge the notification. :rtype: str |
| `/payment/authorize/payment` | jsonrpc | public |  |  | `payment_authorize` | `AuthorizeController.authorize_payment` | Make a payment request and handle the response.  :param str reference: The reference of the transaction :param int partner_id: The partner making the transaction, as a `res.partner` id :param str access_token: The access token used to verify the provided values :param dict opaque_data: The payment d |
| `expression` | http | public | ["POST"] |  | `payment_buckaroo` | `BuckarooController.buckaroo_return_from_checkout` | Process the payment data sent by Buckaroo after redirection from checkout.  The route is flagged with `save_session=False` to prevent the system from assigning a new session to the user if they are redirected to this route with a POST request. Indeed, as the session cookie is created without a `Same |
| `expression` | http | public | ["POST"] |  | `payment_buckaroo` | `BuckarooController.buckaroo_webhook` | Process the payment data sent by Buckaroo to the webhook.  See https://www.pronamic.nl/wp-content/uploads/2013/04/BPE-3.0-Gateway-HTML.1.02.pdf.  :param dict raw_data: The un-formatted payment data :return: An empty string to acknowledge the notification :rtype: str |
| `expression` | http | public | ["POST"] |  | `payment_custom` | `CustomController.custom_process_transaction` |  |
| `expression` | jsonrpc | public |  |  | `payment_demo` | `PaymentDemoController.demo_simulate_payment` | Simulate the response of a payment request.  :param dict data: The simulated payment data. :return: None |
| `expression` | http | public | ["GET"] |  | `payment_dpo` | `DPOController.dpo_return_from_checkout` | Process the payment data sent by DPO after redirection.  :param dict data: The payment data. |
| `expression` | http | public | ["GET", "POST"] |  | `payment_ecpay` | `EcpayController.ecpay_return_from_checkout` | Process the notification data (if included) sent by ECPay after redirection.  The route is flagged with `save_session=False` to prevent the system from assigning a new session to the user if they are redirected to this route with a POST request. Indeed, as the session cookie is created without a `Sa |
| `expression` | http | public | ["POST"] |  | `payment_ecpay` | `EcpayController.ecpay_webhook` | Process the payment data sent by ECPay to the webhook.  The route is flagged with `save_session=False` to prevent the system from assigning a new session to the user.  :param dict data: The payment data. :return: The '1\|OK' string to acknowledge the notification. :rtype: str |
| `expression` | http | public | ["GET"] |  | `payment_flutterwave` | `FlutterwaveController.flutterwave_return_from_checkout` | Process the payment data sent by Flutterwave after redirection from checkout.  :param dict data: The payment data. |
| `expression` | http | public | ["GET"] |  | `payment_flutterwave` | `FlutterwaveController.flutterwave_return_from_authorization` | Process the response sent by Flutterwave after authorization.  :param str response: The stringified JSON response. |
| `expression` | http | public | ["POST"] |  | `payment_flutterwave` | `FlutterwaveController.flutterwave_webhook` | Process the payment data sent by Flutterwave to the webhook.  :return: An empty string to acknowledge the notification. :rtype: str |
| `expression` | http | public | ["POST"] |  | `payment_iyzico` | `IyzicoController.iyzico_return_from_payment` | Process the payment data sent by Iyzico after redirection from checkout.  The route is flagged with `save_session=False` to prevent the system from assigning a new session to the user if they are redirected to this route with a POST request. Indeed, as the session cookie is created without a `SameSi |
| `expression` | http | public | ["POST"] |  | `payment_iyzico` | `IyzicoController.iyzico_webhook` | Process the payment data sent by Iyzico to the webhook.  See https://docs.iyzico.com/en/advanced/webhook.  :return: An empty response to acknowledge the notification. :rtype: the system.http.Response |
| `expression` | http | user | ["GET"] | True | `payment_mercado_pago` | `MercadoPagoOnboardingController.mercado_pago_return_from_authorization` | Exchange the authorization code for an access token and redirect to the provider form.  :param dict data: The authorization code received from Mercado Pago, in addition to the                   provided provider id and CSRF token that were sent back by the proxy. :raise Forbidden: If the received CS |
| `/payment/mercado_pago/payments` | jsonrpc | public |  |  | `payment_mercado_pago` | `MercadoPagoPaymentController.mercado_pago_payment` | Make a payment request and process the payment data.  :param str reference: The reference of the transaction :param int transaction_amount: The amount of the transaction in minor units of the currency :param int token: The transaction token of card received from Mercado Pago. :param int installments |
| `expression` | http | public | ["GET"] |  | `payment_mercado_pago` | `MercadoPagoPaymentController.mercado_pago_return_from_checkout` | Process the payment data sent by Mercado Pago after redirection from checkout.  :param dict data: The payment data. |
| `expression` | http | public | ["POST"] |  | `payment_mercado_pago` | `MercadoPagoPaymentController.mercado_pago_webhook` | Process the payment data sent by Mercado Pago to the webhook.  :param str reference: The transaction reference embedded in the webhook URL. :param dict _kwargs: The extra query parameters. :return: An empty string to acknowledge the notification. :rtype: str |
| `expression` | http | public | ["GET", "POST"] |  | `payment_mollie` | `MollieController.mollie_return_from_checkout` | Process the payment data sent by Mollie after redirection from checkout.  The route is flagged with `save_session=False` to prevent the system from assigning a new session to the user if they are redirected to this route with a POST request. Indeed, as the session cookie is created without a `SameSi |
| `expression` | http | public | ["POST"] |  | `payment_mollie` | `MollieController.mollie_webhook` | Process the payment data sent by Mollie to the webhook.  :param dict data: The payment data (only `id`) and the transaction reference (`ref`)                   embedded in the return URL :return: An empty string to acknowledge the notification :rtype: str |
| `expression` | http | public | ["GET"] |  | `payment_nuvei` | `NuveiController.nuvei_return_from_checkout` | Process the payment data sent by Nuvei after redirection.  :param str tx_ref: The optional reference of the transaction having been canceled/errored. :param str error_access_token: The optional access token to verify the authenticity of                                requests for errored payments. : |
| `expression` | http | public | ["POST"] |  | `payment_nuvei` | `NuveiController.nuvei_webhook` | Process the payment data sent by Nuvei to the webhook.  See https://docs.nuvei.com/documentation/integration/webhooks/payment-dmns/.  :param dict data: The payment data. :return: The 'OK' string to acknowledge the notification. :rtype: str |
| `expression` | http | public | ["GET"] |  | `payment_paymob` | `PaymobController.paymob_return_from_checkout` | Process the payment data sent by Paymob after redirection from checkout.  :param dict data: The payment data. |
| `expression` | http | public | ["POST"] |  | `payment_paymob` | `PaymobController.paymob_webhook` | Process the payment data sent by Paymob to the webhook.  :param dict data: The payment data. :return: An empty string to acknowledge the notification. :rtype: str |
| `expression` | jsonrpc | public | ["POST"] |  | `payment_paypal` | `PaypalController.paypal_complete_order` | Make a capture request and process the payment data.  :param string order_id: The order id provided by PayPal to identify the order. :param str reference: The reference of the transaction, used to generate the idempotency                       key. :return: None |
| `expression` | http | public | ["POST"] |  | `payment_paypal` | `PaypalController.paypal_webhook` | Process the payment data sent by PayPal to the webhook.  See https://developer.paypal.com/docs/api/webhooks/v1/.  :return: An empty string to acknowledge the notification. :rtype: str |
| `expression` | http | public | ["POST"] |  | `payment_payu` | `PayuController.payu_return_from_checkout` | Process the payment data sent by PayU after redirection from checkout.  The route is flagged with `save_session=False` to prevent the system from assigning a new session to the user if they are redirected to this route with a POST request. Indeed, as the session cookie is created without a `SameSite |
| `expression` | http | public | ["POST"] |  | `payment_payu` | `PayuController.payu_webhook` | Process the payment data sent by PayU through the webhook.  :return: An empty response to acknowledge the notification. :rtype: the system.http.Response |
| `expression` | http | user | ["GET"] | True | `payment_payu` | `PayUOnboardingController.payu_return_from_authorization` | Handle the PayU OAuth callback.  :param dict data: The authorization code and merchant ID received from PayU, in addition to                   the the system provider id and CSRF token sent back by the proxy :raise Forbidden: If the received CSRF token cannot be verified :raise ValidationError: If t |
| `expression` | http | public | ["POST"] |  | `payment_razorpay` | `RazorpayController.razorpay_return_from_checkout` | Process the payment data sent by Razorpay after redirection from checkout.  The route is configured with save_session=False to prevent the system from creating a new session when the user is redirected here via a POST request. Indeed, as the session cookie is created without a `SameSite` attribute,  |
| `expression` | http | public | ["POST"] |  | `payment_razorpay` | `RazorpayController.razorpay_webhook` | Process the payment data sent by Razorpay to the webhook.  :return: An empty string to acknowledge the notification. :rtype: str |
| `expression` | http | user | ["GET"] | True | `payment_razorpay` | `RazorpayController.razorpay_return_from_authorization` | Exchange the authorization code for an access token and redirect to the provider form.  :param dict data: The authorization code received from Razorpay, in addition to the provided                   provider id and CSRF token that were sent back by the proxy. :raise Forbidden: If the received CSRF t |
| `expression` | http | public | ["GET"] |  | `payment_redsys` | `RedsysController.redsys_return_from_checkout` | Process the payment data sent by Redsys after redirection.  :param dict encoded_data: The encoded payment data. |
| `expression` | http | public | ["POST"] |  | `payment_redsys` | `RedsysController.redsys_webhook` | Process the payment data sent by Redsys to the webhook.  :param dict encoded_data: The encoded payment data. :return: The 'OK' string to acknowledge the notification. :rtype: str |
| `expression` | http | public | ["GET"] |  | `payment_stripe` | `StripeController.stripe_return` | Process the payment data sent by Stripe after redirection from payment.  Customers go through this route regardless of whether the payment was direct or with redirection to Stripe or to an external service (e.g., for strong authentication).  :param dict data: The payment data, including the referenc |
| `expression` | http | public | ["POST"] |  | `payment_stripe` | `StripeController.stripe_webhook` | Process the payment data sent by Stripe to the webhook.  :return: An empty string to acknowledge the notification. :rtype: str |
| `expression` | http | public |  |  | `payment_stripe` | `StripeController.stripe_apple_pay_get_domain_association_file` | Get the domain association file for Stripe's Apple Pay.  Stripe handles the process of "merchant validation" described in Apple's documentation for Apple Pay on the Web. Stripe and Apple will access this route to check the content of the file and verify that the web domain is registered.  See https: |
| `expression` | http | user | ["GET"] |  | `payment_stripe` | `OnboardingController.stripe_return_from_onboarding` | Redirect the user to the provider form of the onboarded Stripe account.  The user is redirected to this route by Stripe after or during (if the user clicks on a dedicated button) the onboarding.  :param str provider_id: The provider linked to the Stripe account being onboarded, as a                  |
| `expression` | http | user | ["GET"] |  | `payment_stripe` | `OnboardingController.stripe_refresh_onboarding` | Redirect the user to a new Stripe Connect onboarding link.  The user is redirected to this route by Stripe if the onboarding link they used was expired.  :param str provider_id: The provider linked to the Stripe account being onboarded, as a                         `payment.provider` id :param str a |
| `expression` | http | public | ["GET"] |  | `payment_toss_payments` | `TossPaymentsController._toss_payments_success_return` | Process the payment data after redirection from successful payment.  :param dict data: The payment data. Expected keys: orderId, paymentKey, amount. |
| `expression` | http | public | ["GET"] |  | `payment_toss_payments` | `TossPaymentsController._toss_payments_failure_return` | Process the payment data after redirection from failed payment.  Note: The access token is used to verify the request is coming from Toss Payments since we don't have paymentKey in the failure return URL to verify the request via API call. :param dict data: The payment data. Expected keys: access_to |
| `expression` | http | public | ["POST"] |  | `payment_toss_payments` | `TossPaymentsController._toss_payments_webhook` | Process the event data sent to the webhook.  See https://docs.tosspayments.com/reference/using-api/webhook-events#%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%B3%B8%EB%AC%B8 for the event message schema.  :return: An empty string to acknowledge the notification. :rtype: str |
| `expression` | http | public | ["GET"] |  | `payment_worldline` | `WorldlineController.worldline_return_from_checkout` | Process the payment data sent by Worldline after redirection.  :param dict data: The payment data, including the provider id appended to the URL in                   `_get_specific_rendering_values`. |
| `expression` | http | public | ["POST"] |  | `payment_worldline` | `WorldlineController.worldline_webhook` | Process the payment data sent by Worldline to the webhook.  See https://docs.direct.worldline-solutions.com/en/integration/api-developer-guide/webhooks.  :return: An empty string to acknowledge the notification. :rtype: str |
| `/payment/xendit/payment` | jsonrpc | public |  |  | `payment_xendit` | `XenditController.xendit_payment` | Make a payment by token request and handle the response.  :param str reference: The reference of the transaction. :param str token_ref: The reference of the Xendit token to use to make the payment. :param str access_token: The access token used to verify the provided values :param str auth_id: The a |
| `expression` | http | public | ["POST"] |  | `payment_xendit` | `XenditController.xendit_webhook` | Process the payment data sent by Xendit to the webhook.  :return: The 'accepted' string to acknowledge the notification. |
| `expression` | http | public | ["GET"] |  | `payment_xendit` | `XenditController.xendit_return` | Set draft transaction to pending after successfully returning from Xendit. |
| `/pos_customer_display/<id_>/<device_uuid>` | http | public |  | True | `point_of_sale` | `PosCustomerDisplay.pos_customer_display` |  |
| `/pos/service-worker.js` | http | user |  |  | `point_of_sale` | `PosController.pos_web_service_worker` |  |
| `/pos/web` | http | user |  |  | `point_of_sale` | `PosController.old_pos_web` |  |
| `/pos/ui` | http | user |  |  | `point_of_sale` | `PosController.old_pos_web` |  |
| `/pos/ui/<config_id>` | http | user |  |  | `point_of_sale` | `PosController.pos_web` | Open a pos session for the given config.  The right pos session will be selected to open, if non is open yet a new session will be created.  /pos/ui and /pos/web both can be used to access the POS. On the SaaS, /pos/ui uses HTTPS while /pos/web uses HTTP.  :param debug: The debug mode to load the se |
| `/pos/ui/<config_id>/<path:subpath>` | http | user |  |  | `point_of_sale` | `PosController.pos_web` | Open a pos session for the given config.  The right pos session will be selected to open, if non is open yet a new session will be created.  /pos/ui and /pos/web both can be used to access the POS. On the SaaS, /pos/ui uses HTTPS while /pos/web uses HTTP.  :param debug: The debug mode to load the se |
| `/pos/ping` | jsonrpc | user |  |  | `point_of_sale` | `PosController.pos_ping` |  |
| `/pos/sale_details_report` | http | user |  |  | `point_of_sale` | `PosController.print_sale_details` |  |
| `/pos/ticket` | http | public |  | True | `point_of_sale` | `PosController.invoice_request_screen` |  |
| `/pos/ticket/validate` | http | public |  | True | `point_of_sale` | `PosController.show_ticket_validation_screen` |  |
| `/web/image/pos.config/<id>/<string:field>` | http | public |  |  | `point_of_sale` | `PointOfSaleBinary.point_of_sale_content_image` |  |
| `/web/image/pos.config/<id>/<string:field>/<int:width>x<int:height>` | http | public |  |  | `point_of_sale` | `PointOfSaleBinary.point_of_sale_content_image` |  |
| `/mail/unfollow` | http | user |  | True | `portal` | `MailController.mail_action_unfollow` |  |
| `/my/counters` | jsonrpc | user |  | True | `portal` | `CustomerPortal.counters` |  |
| `/my` | http | user |  | True | `portal` | `CustomerPortal.home` |  |
| `/my/home` | http | user |  | True | `portal` | `CustomerPortal.home` |  |
| `/my/account` | http | user |  | True | `portal` | `CustomerPortal.account` |  |
| `/my/addresses` | http | user |  | True | `portal` | `CustomerPortal.my_addresses` | Display the user's addresses. |
| `/my/address` | http | user | ["GET"] | True | `portal` | `CustomerPortal.portal_address` | Display the address form.  A partner and/or an address type can be given through the query string params to specify which address to update or create, and its type.  :param str partner_id: The partner to update with the address form, if any, as a     `res.partner` id. :param str address_type: The ty |
| `/my/address/submit` | http | user | ["POST"] | True | `portal` | `CustomerPortal.portal_address_submit` | Create or update an address from portal and redirect to appropriate page.  If it succeeds, it returns the URL to redirect (client-side) to. If it fails (missing or invalid information), it highlights the problematic form input with the appropriate error message.  :param str partner_id: The partner w |
| `/my/address/country_info/<model("res.country"):country>` | jsonrpc | public | ["POST"] | True | `portal` | `CustomerPortal.portal_address_country_info` |  |
| `/my/address/archive` | jsonrpc | user | ["POST"] | True | `portal` | `CustomerPortal.address_archive` |  |
| `/my/security` | http | user | ["GET", "POST"] | True | `portal` | `CustomerPortal.security` |  |
| `/my/deactivate_account` | http | user | ["POST"] | True | `portal` | `CustomerPortal.deactivate_account` |  |
| `/portal/attachment/remove` | jsonrpc | public |  |  | `portal` | `CustomerPortal.attachment_remove` | Remove the given `attachment_id`, only if it is in a "pending" state.  The user must have access right on the attachment or provide a valid `access_token`. |
| `/mail/avatar/mail.message/<int:res_id>/author_avatar/<int:width>x<int:height>` | http | public |  |  | `portal` | `PortalChatter.portal_avatar` | Get the avatar image in the chatter of the portal |
| `/portal/chatter_init` | jsonrpc | public |  | True | `portal` | `PortalChatter.portal_chatter_init` |  |
| `/mail/chatter_fetch` | jsonrpc | public |  | True | `portal` | `PortalChatter.portal_message_fetch` |  |
| `/mail/update_is_internal` | jsonrpc | user |  | True | `portal` | `PortalChatter.portal_message_update_is_internal` |  |
| `/website/rating/comment` | jsonrpc | user | ["POST"] | True | `portal_rating` | `PortalRating.publish_rating_comment` |  |
| `/pos_adyen/notification` | jsonrpc | public | ["POST"] |  | `pos_adyen` | `PosAdyenController.notification` |  |
| `/pos_mercado_pago/notification` | http | none | ["POST"] |  | `pos_mercado_pago` | `PosMercadoPagoWebhook.notification` | Process the notification sent by Mercado Pago  Notification format is always json |
| `/pos_mollie/webhook` | http | public | ["POST"] |  | `pos_mollie` | `PosMollie.mollie_webhook` |  |
| `/pos/pay/<int:pos_order_id>` | http | public | ["GET"] | True | `pos_online_payment` | `PaymentPortal.pos_order_pay` | Behaves like payment.PaymentPortal.payment_pay but for POS online payment.  :param int pos_order_id: The POS order to pay, as a `pos.order` id :param str access_token: The access token used to verify the user :param str exit_route: The URL to open to leave the POS online payment flow  :return: The r |
| `/pos/pay/transaction/<int:pos_order_id>` | jsonrpc | public |  | True | `pos_online_payment` | `PaymentPortal.pos_order_pay_transaction` | Behaves like payment.PaymentPortal.payment_transaction but for POS online payment.  :param int pos_order_id: The POS order to pay, as a `pos.order` id :param str access_token: The access token used to verify the user :param str exit_route: The URL to open to leave the POS online payment flow :param  |
| `/pos/pay/confirmation/<int:pos_order_id>` | http | public | ["GET"] | True | `pos_online_payment` | `PaymentPortal.pos_order_pay_confirmation` | Behaves like payment.PaymentPortal.payment_confirm but for POS online payment.  :param int pos_order_id: The POS order to confirm, as a `pos.order` id :param str tx_id: The transaction to confirm, as a `payment.transaction` id :param str access_token: The access token used to verify the user :param  |
| `/qfpay/notify` | http | public | ["POST"] |  | `pos_qfpay` | `QFPayNotificationController.qfpay_notify` | Endpoint to receive QFPay asynchronous payment/refund notifications. Verifies signature and returns 'SUCCESS' if valid, else 400 error. |
| `/pos_safaricom/callback` | http | public | ["POST"] |  | `pos_safaricom` | `SafaricomController.safaricom_callback` | Handle M-Pesa STK Push callback |
| `/c2b/validation/callback` | http | public | ["POST"] |  | `pos_safaricom` | `SafaricomController.c2b_validation_callback` | Validate the payment before charging the customer For now, we accept all payments (ResponseType is set to 'Completed') |
| `/c2b/confirmation/callback` | http | public | ["POST"] |  | `pos_safaricom` | `SafaricomController.c2b_confirmation_callback` | Handle C2B payment confirmation via Lipa na M-PESA |
| `/pos-self-order/process-order/<device_type>/` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.process_order` |  |
| `/pos-self-order/get-order/<int:order_id>` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.get_order` |  |
| `/pos-self-order/validate-partner` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.validate_partner` |  |
| `/pos-self-order/remove-order` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.remove_order` |  |
| `/pos-self-order/send_self_order_receipt` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.send_self_order_receipt` |  |
| `/pos-self-order/get-user-data` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.get_orders_by_access_token` |  |
| `/kiosk/payment/<int:pos_config_id>/<device_type>` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.pos_self_order_kiosk_payment` |  |
| `/pos_self_order/kiosk/increment_nb_print/` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.pos_kiosk_increment_nb_print` |  |
| `/pos-self-order/change-printer-status` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.change_printer_status` |  |
| `/pos-self-order/get-slots` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfOrderController.get_slots` |  |
| `/pos-self/ping` | jsonrpc | public |  |  | `pos_self_order` | `PosSelfOrderController.pos_ping` |  |
| `/pos-self/<config_id>` | http | public |  | True | `pos_self_order` | `PosSelfKiosk.start_self_ordering` |  |
| `/pos-self/<config_id>/<path:subpath>` | http | public |  | True | `pos_self_order` | `PosSelfKiosk.start_self_ordering` |  |
| `/pos-self/data/<config_id>` | jsonrpc | public |  | True | `pos_self_order` | `PosSelfKiosk.get_self_ordering_data` |  |
| `/pos-self/relations/<config_id>` | jsonrpc | public |  |  | `pos_self_order` | `PosSelfKiosk.get_self_ordering_relations` |  |
| `/pos-self-order/pine-labs-fetch-payment-status/` | jsonrpc | public |  |  | `pos_self_order_pine_labs` | `PosSelfOrderPineLabsController.pine_labs_fetch_payment_status` |  |
| `/pos-self-order/pine-labs-cancel-transaction/` | jsonrpc | public |  |  | `pos_self_order_pine_labs` | `PosSelfOrderPineLabsController.pine_labs_cancel_transaction` |  |
| `/pos-self-order/razorpay-fetch-payment-status/` | jsonrpc | public |  | True | `pos_self_order_razorpay` | `PosSelfOrderControllerRazorpay.razorpay_payment_status` |  |
| `/pos-self-order/razorpay-cancel-transaction/` | jsonrpc | public |  | True | `pos_self_order_razorpay` | `PosSelfOrderControllerRazorpay.razorpay_cancel_status` |  |
| `/pos-self-order/stripe-connection-token/` | jsonrpc | public |  | True | `pos_self_order_stripe` | `PosSelfOrderControllerStripe.get_stripe_creditentials` |  |
| `/pos-self-order/stripe-capture-payment/` | jsonrpc | public |  | True | `pos_self_order_stripe` | `PosSelfOrderControllerStripe.stripe_capture_payment` |  |
| `/pos_self_order_viva_com/poll_payment` | jsonrpc | public |  |  | `pos_self_order_viva_com` | `PosSelfOrderVivaComController.poll_payment` |  |
| `/pos_viva_com/notification` | http | none |  |  | `pos_viva_com` | `PosVivaComController.notification` |  |
| `/product/catalog/order_lines_info` | jsonrpc | user |  |  | `product` | `ProductCatalogController.product_catalog_get_order_lines_info` | Returns products information to be shown in the catalog.  :param string res_model: The order model. :param int order_id: The order id. :param list product_ids: The products currently displayed in the product catalog, as a list                          of `product.product` ids. :rtype: dict :return:  |
| `/product/catalog/update_order_line_info` | jsonrpc | user |  |  | `product` | `ProductCatalogController.product_catalog_update_order_line_info` | Update order line information on a given order for a given product.  :param string res_model: The order model. :param int order_id: The order id. :param int product_id: The product, as a `product.product` id. :return: The unit price price of the product, based on the pricelist of the order and       |
| `/product/export/pricelist/` | http | user |  |  | `product` | `ProductPricelistExportController.export_pricelist` |  |
| `/product/document/upload` | http | user | ["POST"] |  | `product` | `ProductDocumentController.upload_document` |  |
| `/my/projects` | http | user |  | True | `project` | `ProjectCustomerPortal.portal_my_projects` |  |
| `/my/projects/page/<int:page>` | http | user |  | True | `project` | `ProjectCustomerPortal.portal_my_projects` |  |
| `/my/projects/<int:project_id>` | http | public |  | True | `project` | `ProjectCustomerPortal.portal_my_project` |  |
| `/my/projects/<int:project_id>/page/<int:page>` | http | public |  | True | `project` | `ProjectCustomerPortal.portal_my_project` |  |
| `/my/projects/<int:project_id>/project_sharing` | http | user | ["GET"] |  | `project` | `ProjectCustomerPortal.render_project_backend_view` |  |
| `/my/projects/<int:project_id>/project_sharing/<path:subpath>` | http | user | ["GET"] |  | `project` | `ProjectCustomerPortal.render_project_backend_view` |  |
| `/my/projects/<int:project_id>/task/<int:task_id>` | http | public |  | True | `project` | `ProjectCustomerPortal.portal_my_project_task` |  |
| `/my/projects/<int:project_id>/task/<int:task_id>/subtasks` | http | user | ["GET"] | True | `project` | `ProjectCustomerPortal.portal_my_project_subtasks` |  |
| `/my/projects/<int:project_id>/task/<int:task_id>/recurrent_tasks` | http | user | ["GET"] | True | `project` | `ProjectCustomerPortal.portal_my_project_recurrent_tasks` |  |
| `/my/tasks` | http | user |  | True | `project` | `ProjectCustomerPortal.portal_my_tasks` |  |
| `/my/tasks/page/<int:page>` | http | user |  | True | `project` | `ProjectCustomerPortal.portal_my_tasks` |  |
| `/my/tasks/<int:task_id>` | http | public |  | True | `project` | `ProjectCustomerPortal.portal_my_task` |  |
| `/project_sharing/attachment/add_image` | http | user | ["POST"] | True | `project` | `ProjectCustomerPortal.add_image` |  |
| `/mail_plugin/project/search` | jsonrpc | outlook |  |  | `project_mail_plugin` | `ProjectClient.projects_search` | Used in the plugin side when searching for projects. Fetches projects that have names containing the search_term. |
| `/mail_plugin/task/create` | jsonrpc | outlook |  |  | `project_mail_plugin` | `ProjectClient.task_create` |  |
| `/mail_plugin/project/create` | jsonrpc | outlook |  |  | `project_mail_plugin` | `ProjectClient.project_create` |  |
| `/my/rfq` | http | user |  | True | `purchase` | `CustomerPortal.portal_my_requests_for_quotation` |  |
| `/my/rfq/page/<int:page>` | http | user |  | True | `purchase` | `CustomerPortal.portal_my_requests_for_quotation` |  |
| `/my/purchase` | http | user |  | True | `purchase` | `CustomerPortal.portal_my_purchase_orders` |  |
| `/my/purchase/page/<int:page>` | http | user |  | True | `purchase` | `CustomerPortal.portal_my_purchase_orders` |  |
| `/my/purchase/<int:order_id>` | http | public |  | True | `purchase` | `CustomerPortal.portal_my_purchase_order` |  |
| `/my/purchase/<int:order_id>/update` | jsonrpc | public |  | True | `purchase` | `CustomerPortal.portal_my_purchase_order_update_dates` | User update scheduled date on purchase order line. |
| `/my/purchase/<int:order_id>/download_edi` | http | public |  | True | `purchase` | `CustomerPortal.portal_my_purchase_order_download_edi` | An endpoint to download EDI file representation. |
| `/rate/<string:token>/<int:rate>` | http | public |  | True | `rating` | `Rating.action_open_rating` |  |
| `/rate/<string:token>/submit_feedback` | http | public | ["post", "get"] | True | `rating` | `Rating.action_submit_rating` |  |
| `/web/version` | http | none |  |  | `rpc` | `RPC.version` |  |
| `/json/version` | http | none |  |  | `rpc` | `RPC.version` |  |
| `/json/2` | json2 | public | ["GET", "POST", "PUT", "DELETE", "PATCH"] |  | `rpc` | `WebJson2Controller.web_json_2_404` |  |
| `/json/2/<path:subpath>` | json2 | public | ["GET", "POST", "PUT", "DELETE", "PATCH"] |  | `rpc` | `WebJson2Controller.web_json_2_404` |  |
| `/json/2/<__model__>/<__method__>` | json2 | bearer | ["POST"] |  | `rpc` | `WebJson2Controller.web_json_2_rpc` |  |
| `/jsonrpc` | jsonrpc | none |  |  | `rpc` | `JSONRPC.jsonrpc` | Method used by client APIs to contact the system. |
| `/xmlrpc/<service>` | http | none | ["POST"] |  | `rpc` | `XMLRPC.xmlrpc_1` | XML-RPC service that returns faultCode as strings.  This entrypoint is historical and non-compliant, but kept for backwards-compatibility. |
| `/xmlrpc/2/<service>` | http | none | ["POST"] |  | `rpc` | `XMLRPC.xmlrpc_2` | XML-RPC service that returns faultCode as int. |
| `/sale/combo_configurator/get_data` | jsonrpc | user |  |  | `sale` | `SaleComboConfiguratorController.sale_combo_configurator_get_data` | Return data about the specified combo product.  :param int product_tmpl_id: The product for which to get data, as a `product.template` id. :param int quantity: The quantity of the product. :param str date: The date to use to compute prices. :param int\|None currency_id: The currency to use to comput |
| `/sale/combo_configurator/get_price` | jsonrpc | user |  |  | `sale` | `SaleComboConfiguratorController.sale_combo_configurator_get_price` | Return the price of the specified combo product.  :param int product_tmpl_id: The product for which to get the price, as a `product.template`     id. :param int quantity: The quantity of the product. :param str date: The date to use to compute the price. :param int\|None currency_id: The currency to |
| `/my/quotes` | http | user |  | True | `sale` | `CustomerPortal.portal_my_quotes` |  |
| `/my/quotes/page/<int:page>` | http | user |  | True | `sale` | `CustomerPortal.portal_my_quotes` |  |
| `/my/orders` | http | user |  | True | `sale` | `CustomerPortal.portal_my_orders` |  |
| `/my/orders/page/<int:page>` | http | user |  | True | `sale` | `CustomerPortal.portal_my_orders` |  |
| `/my/orders/<int:order_id>` | http | public |  | True | `sale` | `CustomerPortal.portal_order_page` |  |
| `/my/orders/<int:order_id>/accept` | jsonrpc | public |  | True | `sale` | `CustomerPortal.portal_quote_accept` |  |
| `/my/orders/<int:order_id>/decline` | http | public | ["POST"] | True | `sale` | `CustomerPortal.portal_quote_decline` |  |
| `/my/orders/<int:order_id>/document/<int:document_id>` | http | public |  |  | `sale` | `CustomerPortal.portal_quote_document` |  |
| `/my/orders/<int:order_id>/download_edi` | http | public |  | True | `sale` | `CustomerPortal.portal_my_sale_order_download_edi` | An endpoint to download EDI file representation. |
| `/my/orders/<int:order_id>/transaction` | jsonrpc | public |  |  | `sale` | `PaymentPortal.portal_order_transaction` | Create a draft transaction and return its processing values.  :param int order_id: The sales order to pay, as a `sale.order` id :param str access_token: The access token used to authenticate the request :param dict kwargs: Locally unused data passed to `_create_transaction` :return: The mandatory va |
| `/sale/product_configurator/get_values` | jsonrpc | user |  |  | `sale` | `SaleProductConfiguratorController.sale_product_configurator_get_values` | Return all product information needed for the product configurator.  :param int product_template_id: The product for which to seek information, as a     `product.template` id. :param int quantity: The quantity of the product. :param int currency_id: The currency of the transaction, as a `res.currenc |
| `/sale/product_configurator/create_product` | jsonrpc | user | ["POST"] |  | `sale` | `SaleProductConfiguratorController.sale_product_configurator_create_product` | Create the product when there is a dynamic attribute in the combination.  :param int product_template_id: The product for which to seek information, as a     `product.template` id. :param list(int) ptav_ids: The combination of the product, as a list of     `product.template.attribute.value` ids. :rt |
| `/sale/product_configurator/update_combination` | jsonrpc | user | ["POST"] |  | `sale` | `SaleProductConfiguratorController.sale_product_configurator_update_combination` | Return the updated combination information.  :param int product_template_id: The product for which to seek information, as a     `product.template` id. :param list(int) ptav_ids: The combination of the product, as a list of     `product.template.attribute.value` ids. :param int currency_id: The curr |
| `/sale/product_configurator/get_optional_products` | jsonrpc | user |  |  | `sale` | `SaleProductConfiguratorController.sale_product_configurator_get_optional_products` | Return information about optional products for the given `product.template`.  :param int product_template_id: The product for which to seek information, as a     `product.template` id. :param list(int) ptav_ids: The combination of the product, as a list of     `product.template.attribute.value` ids. |
| `expression` | http | public | ["POST"] |  | `sale_gelato` | `GelatoController.gelato_webhook` | Process the notification data sent by Gelato to the webhook.  See https://dashboard.gelato.com/docs/orders/order_details/#order-statuses for the event codes.  :return: An empty response to acknowledge the notification. :rtype: the system.http.Response |
| `/my/orders/<int:order_id>/update_line_dict` | jsonrpc | public |  | True | `sale_management` | `CustomerPortal.portal_quote_option_update` | Update the quantity of an optional SOline from a SO.  :param int order_id: `sale.order` id :param int line_id: `sale.order.line` id :param str access_token: portal access_token of the specified order :param bool remove: if true, 1 unit will be removed from the line :param float input_quantity: if sp |
| `/sale_pdf_quote_builder/quotation_document/upload` | http | user | ["POST"] |  | `sale_pdf_quote_builder` | `QuotationDocumentController.upload_document` |  |
| `/my/picking/pdf/<int:picking_id>` | http | public |  | True | `sale_stock` | `SaleStockPortal.portal_my_picking_report` | Print delivery slip for customer, using either access rights or access token to be sure customer has access |
| `/my/picking/return/pdf/<int:picking_id>` | http | public |  | True | `sale_stock` | `SaleStockPortal.portal_my_picking_return_report` | Print return label for customer, using either access rights or access token to be sure customer has access |
| `/my/tasks/<task_id>/orders/invoices` | http | user |  | True | `sale_timesheet` | `PortalProjectAccount.portal_my_tasks_invoices` |  |
| `/my/tasks/<task_id>/orders/invoices/page/<int:page>` | http | user |  | True | `sale_timesheet` | `PortalProjectAccount.portal_my_tasks_invoices` |  |
| `/sms/status` | jsonrpc | public |  |  | `sms` | `SmsController.update_sms_status` | Receive a batch of delivery reports from IAP  :param message_statuses:     [         {             'sms_status': status0,             'uuids': [uuid00, uuid01, ...],         }, {             'sms_status': status1,             'uuids': [uuid10, uuid11, ...],         },         ...     ] |
| `/sms_twilio/status/<string:uuid>` | http | public | ["POST"] |  | `sms_twilio` | `SmsTwilioController.update_sms_status` |  |
| `/spreadsheet/log` | jsonrpc | user | ["POST"] |  | `spreadsheet` | `SpreadsheetController.log_action` |  |
| `/spreadsheet/dashboard/data/<model("spreadsheet.dashboard"):dashboard>` | http | user |  |  | `spreadsheet_dashboard` | `DashboardDataRoute.get_dashboard_data` |  |
| `/dashboard/share/<int:share_id>/<token>` | http | public |  |  | `spreadsheet_dashboard` | `DashboardShareRoute.share_portal` |  |
| `/dashboard/download/<int:share_id>/<token>` | http | user |  |  | `spreadsheet_dashboard` | `DashboardShareRoute.download` |  |
| `/dashboard/data/<int:share_id>/<token>` | http | public | ["GET"] |  | `spreadsheet_dashboard` | `DashboardShareRoute.get_shared_dashboard_data` |  |
| `/stock/<string:output_format>/<string:report_name>` | http | user |  |  | `stock` | `StockReportController.report` |  |
| `/survey/test/<string:survey_token>` | http | user |  | True | `survey` | `Survey.survey_test` | Test mode for surveys: create a test answer, only for managers or officers testing their surveys |
| `/survey/retry/<string:survey_token>/<string:answer_token>` | http | public |  | True | `survey` | `Survey.survey_retry` | This route is called whenever the user has attempts left and hits the 'Retry' button after failing the survey. |
| `/survey/start/<string:survey_token>` | http | public |  | True | `survey` | `Survey.survey_start` | Start a survey by providing * a token linked to a survey; * a token linked to an answer or generate a new token if access is allowed; |
| `/survey/<string:survey_token>` | http | public |  | True | `survey` | `Survey.survey_display_page` |  |
| `/survey/<string:survey_token>/<string:answer_token>` | http | public |  | True | `survey` | `Survey.survey_display_page` |  |
| `/survey/<string:survey_token>/get_background_image` | http | public |  | True | `survey` | `Survey.survey_get_background` |  |
| `/survey/<string:survey_token>/<int:section_id>/get_background_image` | http | public |  | True | `survey` | `Survey.survey_section_get_background` |  |
| `/survey/get_question_image/<string:survey_token>/<string:answer_token>/<int:question_id>/<int:suggested_answer_id>` | http | public |  | True | `survey` | `Survey.survey_get_question_image` |  |
| `/survey/begin/<string:survey_token>/<string:answer_token>` | jsonrpc | public |  | True | `survey` | `Survey.survey_begin` | Route used to start the survey user input and display the first survey page. Returns an empty dict for the correct answers and the first page html. |
| `/survey/next_question/<string:survey_token>/<string:answer_token>` | jsonrpc | public |  | True | `survey` | `Survey.survey_next_question` | Method used to display the next survey question in an ongoing session. Triggered on all attendees screens when the host goes to the next question. |
| `/survey/submit/<string:survey_token>/<string:answer_token>` | jsonrpc | public |  | True | `survey` | `Survey.survey_submit` | Submit a page from the survey. This will take into account the validation errors and store the answers to the questions. If the time limit is reached, errors will be skipped, answers will be ignored and survey state will be forced to 'done'. Also returns the correct answers if the scoring type is 's |
| `/survey/print/<string:survey_token>` | http | public |  | True | `survey` | `Survey.survey_print` | Display an survey in printable view; if <answer_token> is set, it will grab the answers of the user_input_id that has <answer_token>. |
| `/survey/<model("survey.survey"):survey>/certification_preview` | http | user |  | True | `survey` | `Survey.show_certification_pdf` |  |
| `/survey/<model("survey.survey"):survey>/get_certification_preview` | http | user | ["GET"] | True | `survey` | `Survey.survey_get_certification_preview` |  |
| `/survey/<int:survey_id>/get_certification` | http | user | ["GET"] | True | `survey` | `Survey.survey_get_certification` | The certification document can be downloaded as long as the user has succeeded the certification |
| `/survey/results/<model("survey.survey"):survey>` | http | user |  | True | `survey` | `Survey.survey_report` | Display survey Results & Statistics for given survey.  New structure: {     'survey': current survey browse record,     'question_and_page_data': see ``SurveyQuestion._prepare_statistics()``,     'survey_data'= see ``SurveySurvey._prepare_statistics()``     'search_filters': [],     'search_finished |
| `/survey/session/manage/<string:survey_token>` | http | user |  | True | `survey` | `UserInputSession.survey_session_manage` | Main route used by the host to 'manager' the session. - If the state of the session is 'ready'   We render a template allowing the host to showcase the different options of the session   and to actually start the session.   If there are no questions, a "void content" is displayed instead to avoid di |
| `/survey/session/next_question/<string:survey_token>` | jsonrpc | user |  | True | `survey` | `UserInputSession.survey_session_next_question` | This route is called when the host goes to the next question of the session.  It's not a regular 'request.render' route because we handle the transition between questions using a AJAX call to be able to display a bioutiful fade in/out effect.  It triggers the next question of the session.  We artifi |
| `/survey/session/results/<string:survey_token>` | jsonrpc | user |  | True | `survey` | `UserInputSession.survey_session_results` | This route is called when the host shows the current question's results.  It's not a regular 'request.render' route because we handle the display of results using an AJAX request to be able to include the results in the currently displayed page. |
| `/survey/session/leaderboard/<string:survey_token>` | jsonrpc | user |  | True | `survey` | `UserInputSession.survey_session_leaderboard` | This route is called when the host shows the current question's attendees leaderboard.  It's not a regular 'request.render' route because we handle the display of the leaderboard using an AJAX request to be able to include the results in the currently displayed page. |
| `/s` | http | public |  | True | `survey` | `UserInputSession.survey_session_code` | Renders the survey session code page route. This page allows the user to enter the session code of the survey. It is mainly used to ease survey access for attendees in session mode. |
| `/s/<string:session_code>` | http | public |  | True | `survey` | `UserInputSession.survey_start_short` | " Redirects to 'survey_start' route using a shortened link & token. Shows an error message if the survey is not valid. This route is used in survey sessions where we need short links for people to type. |
| `/survey/check_session_code/<string:session_code>` | jsonrpc | public |  | True | `survey` | `UserInputSession.survey_check_session_code` | Checks if the given code is matching a survey session_code. If yes, redirect to /s/code route. If not, return error. The user is invited to type again the code. |
| `/web/action/load` | jsonrpc | user |  |  | `web` | `Action.load` |  |
| `/web/action/run` | jsonrpc | user |  |  | `web` | `Action.run` |  |
| `/web/action/load_breadcrumbs` | jsonrpc | user |  |  | `web` | `Action.load_breadcrumbs` |  |
| `/web/filestore/<path:_path>` | http | none |  |  | `web` | `Binary.content_filestore` |  |
| `/web/content` | http | public |  |  | `web` | `Binary.content_common` |  |
| `/web/content/<string:xmlid>` | http | public |  |  | `web` | `Binary.content_common` |  |
| `/web/content/<string:xmlid>/<string:filename>` | http | public |  |  | `web` | `Binary.content_common` |  |
| `/web/content/<int:id>` | http | public |  |  | `web` | `Binary.content_common` |  |
| `/web/content/<int:id>/<string:filename>` | http | public |  |  | `web` | `Binary.content_common` |  |
| `/web/content/<string:model>/<int:id>/<string:field>` | http | public |  |  | `web` | `Binary.content_common` |  |
| `/web/content/<string:model>/<int:id>/<string:field>/<string:filename>` | http | public |  |  | `web` | `Binary.content_common` |  |
| `/web/assets/<string:unique>/<string:filename>` | http | public |  |  | `web` | `Binary.content_assets` |  |
| `/web/image` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<string:xmlid>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<string:xmlid>/<string:filename>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<string:xmlid>/<int:width>x<int:height>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<string:xmlid>/<int:width>x<int:height>/<string:filename>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<string:model>/<int:id>/<string:field>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<string:model>/<int:id>/<string:field>/<string:filename>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<string:model>/<int:id>/<string:field>/<int:width>x<int:height>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<string:model>/<int:id>/<string:field>/<int:width>x<int:height>/<string:filename>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<int:id>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<int:id>/<string:filename>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<int:id>/<int:width>x<int:height>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<int:id>/<int:width>x<int:height>/<string:filename>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<int:id>-<string:unique>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<int:id>-<string:unique>/<string:filename>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<int:id>-<string:unique>/<int:width>x<int:height>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/image/<int:id>-<string:unique>/<int:width>x<int:height>/<string:filename>` | http | public |  |  | `web` | `Binary.content_image` |  |
| `/web/binary/upload_attachment` | http | user |  |  | `web` | `Binary.upload_attachment` |  |
| `/web/binary/company_logo` | http | none |  |  | `web` | `Binary.company_logo` |  |
| `/logo` | http | none |  |  | `web` | `Binary.company_logo` |  |
| `/logo.png` | http | none |  |  | `web` | `Binary.company_logo` |  |
| `/web/sign/get_fonts` | jsonrpc | none |  |  | `web` | `Binary.get_fonts` | This route will return a list of base64 encoded fonts.  Those fonts will be proposed to the user when creating a signature using mode 'auto'.  :return: base64 encoded fonts :rtype: list |
| `/web/sign/get_fonts/<string:fontname>` | jsonrpc | none |  |  | `web` | `Binary.get_fonts` | This route will return a list of base64 encoded fonts.  Those fonts will be proposed to the user when creating a signature using mode 'auto'.  :return: base64 encoded fonts :rtype: list |
| `/web/database/selector` | http | none |  |  | `web` | `Database.selector` |  |
| `/web/database/manager` | http | none |  |  | `web` | `Database.manager` |  |
| `/web/database/create` | http | none | ["POST"] |  | `web` | `Database.create` |  |
| `/web/database/duplicate` | http | none | ["POST"] |  | `web` | `Database.duplicate` |  |
| `/web/database/drop` | http | none | ["POST"] |  | `web` | `Database.drop` |  |
| `/web/database/backup` | http | none | ["POST"] |  | `web` | `Database.backup` |  |
| `/web/database/restore` | http | none | ["POST"] |  | `web` | `Database.restore` |  |
| `/web/database/change_password` | http | none | ["POST"] |  | `web` | `Database.change_password` |  |
| `/web/database/list` | jsonrpc | none |  |  | `web` | `Database.list` | Used by Mobile application for listing database :return: List of databases :rtype: list |
| `/web/dataset/call_kw` | jsonrpc | user |  |  | `web` | `DataSet.call_kw` |  |
| `/web/dataset/call_kw/<path:path>` | jsonrpc | user |  |  | `web` | `DataSet.call_kw` |  |
| `/web/dataset/call_button` | jsonrpc | user |  |  | `web` | `DataSet.call_button` |  |
| `/web/dataset/call_button/<path:path>` | jsonrpc | user |  |  | `web` | `DataSet.call_button` |  |
| `/web/domain/validate` | jsonrpc | user |  |  | `web` | `Domain.validate` | Parse `domain` and verify that it can be used to search on `model` :return: True when the domain is valid, otherwise False :raises ValidationError: if `model` is invalid |
| `/web/export/formats` | jsonrpc | user |  |  | `web` | `Export.formats` | Returns all valid export formats  :returns: for each export format, a pair of identifier and printable name :rtype: [(str, str)] |
| `/web/export/get_fields` | jsonrpc | user |  |  | `web` | `Export.get_fields` |  |
| `/web/export/namelist` | jsonrpc | user |  |  | `web` | `Export.namelist` |  |
| `/web/export/csv` | http | user |  |  | `web` | `CSVExport.web_export_csv` |  |
| `/web/export/xlsx` | http | user |  |  | `web` | `ExcelExport.web_export_xlsx` |  |
| `/` | http | none |  |  | `web` | `Home.index` |  |
| `/web` | http | none |  |  | `web` | `Home.web_client` |  |
| `/odoo` | http | none |  |  | `web` | `Home.web_client` |  |
| `/odoo/<path:subpath>` | http | none |  |  | `web` | `Home.web_client` |  |
| `/scoped_app/<path:subpath>` | http | none |  |  | `web` | `Home.web_client` |  |
| `/web/webclient/load_menus` | http | user | ["GET"] |  | `web` | `Home.web_load_menus` | Loads the menus for the webclient :param lang: language in which the menus should be loaded (only works if language is installed) :return: the menus (including the images in Base64) |
| `/web/login` | http | none |  |  | `web` | `Home.web_login` |  |
| `/web/login_successful` | http | user |  | True | `web` | `Home.login_successful_external_user` | Landing page after successful login for external users (unused when portal is installed). |
| `/web/become` | http | user |  |  | `web` | `Home.switch_to_admin` |  |
| `/web/health` | http | none |  |  | `web` | `Home.health` |  |
| `/robots.txt` | http | none |  |  | `web` | `Home.robots` |  |
| `/json/<path:subpath>` | http | user |  |  | `web` | `WebJsonController.web_json` |  |
| `/json/1/<path:subpath>` | http | bearer |  |  | `web` | `WebJsonController.web_json_1` | Simple JSON representation of the views.  Get the JSON representation of the action/view as it would be shown in the web client for the same /the system `subpath`.  Behaviour: - When, the action resolves to a pair (Action, id), `form` view_type.   Otherwise when it resolves to (Action, None), use th |
| `/web/model/get_definitions` | http | user | ["POST"] |  | `web` | `Model.get_model_definitions` |  |
| `/web/pivot/export_xlsx` | http | user |  |  | `web` | `TableExporter.export_xlsx` |  |
| `/web/set_profiling` | http | public |  |  | `web` | `Profiling.profile` |  |
| `/web/speedscope/<profile>` | http | user |  |  | `web` | `Profiling.speedscope` |  |
| `/web/profile_config/<profile>` | http | user |  |  | `web` | `Profiling.profile_config` |  |
| `/report/<converter>/<reportname>` | http | user |  | True | `web` | `ReportController.report_routes` |  |
| `/report/<converter>/<reportname>/<docids>` | http | user |  | True | `web` | `ReportController.report_routes` |  |
| `/report/barcode` | http | public |  |  | `web` | `ReportController.report_barcode` | Contoller able to render barcode images thanks to reportlab. Samples::      <img t-att-src="'/report/barcode/QR/%s' % o.name"/>     <img t-att-src="'/report/barcode/?barcode_type=%s&amp;value=%s&amp;width=%s&amp;height=%s' %         ('QR', o.name, 200, 200)"/>  :param barcode_type: Accepted types: ' |
| `/report/barcode/<barcode_type>/<path:value>` | http | public |  |  | `web` | `ReportController.report_barcode` | Contoller able to render barcode images thanks to reportlab. Samples::      <img t-att-src="'/report/barcode/QR/%s' % o.name"/>     <img t-att-src="'/report/barcode/?barcode_type=%s&amp;value=%s&amp;width=%s&amp;height=%s' %         ('QR', o.name, 200, 200)"/>  :param barcode_type: Accepted types: ' |
| `/report/download` | http | user |  |  | `web` | `ReportController.report_download` | This function is used by 'action_manager_report.js' in order to trigger the download of a pdf/controller report.  :param data: a javascript array JSON.stringified containg report internal url ([0]) and type [1] :returns: Response with an attachment header |
| `/report/check_wkhtmltopdf` | jsonrpc | user |  |  | `web` | `ReportController.check_wkhtmltopdf` |  |
| `/web/session/get_session_info` | jsonrpc | user |  |  | `web` | `Session.get_session_info` |  |
| `/web/session/authenticate` | jsonrpc | none |  |  | `web` | `Session.authenticate` |  |
| `/web/session/get_lang_list` | jsonrpc | none |  |  | `web` | `Session.get_lang_list` |  |
| `/web/session/modules` | jsonrpc | user |  |  | `web` | `Session.modules` |  |
| `/web/session/check` | jsonrpc | user |  |  | `web` | `Session.check` |  |
| `/web/session/account` | jsonrpc | user |  |  | `web` | `Session.account` |  |
| `/web/session/destroy` | jsonrpc | user |  |  | `web` | `Session.destroy` |  |
| `/web/session/logout` | http | none |  |  | `web` | `Session.logout` |  |
| `/web_enterprise/partner/<model("res.partner"):partner>/vcard` | http | user |  |  | `web` | `Partner.download_vcard` |  |
| `/web/partner/vcard` | http | user |  |  | `web` | `Partner.download_vcard` |  |
| `/web/view/edit_custom` | jsonrpc | user |  |  | `web` | `View.edit_custom` | Edit a custom view  :param int custom_id: the id of the edited custom view :param str arch: the edited arch of the custom view :returns: dict with acknowledged operation (result set to True) |
| `/web/webclient/bootstrap_translations` | jsonrpc | none |  |  | `web` | `WebClient.bootstrap_translations` | Load local translations from *.po files, as a temporary solution until we have established a valid session. This is meant only for translating the login page and db management chrome, using the browser's language. |
| `/web/webclient/translations` | http | public |  |  | `web` | `WebClient.translations` | Load the translations for the specified language and modules  :param hash: translations hash, which identifies a version of translations. This method only returns translations if their hash differs from the received one :param mods: the modules, a comma separated list :param lang: the language of th |
| `/web/webclient/version_info` | jsonrpc | none |  |  | `web` | `WebClient.version_info` |  |
| `/web/tests` | http | user |  |  | `web` | `WebClient.unit_tests_suite` |  |
| `/web/tests/legacy` | http | user |  |  | `web` | `WebClient.test_suite` |  |
| `/web/bundle/<string:bundle_name>` | http | public | ["GET"] |  | `web` | `WebClient.bundle` | Request the definition of a bundle, including its javascript and css bundled assets |
| `/web/manifest.webmanifest` | http | public | ["GET"] |  | `web` | `WebManifest.webmanifest` | Returns a WebManifest describing the metadata associated with a web application. Using this metadata, user agents can provide developers with means to create user experiences that are more comparable to that of a native application. |
| `/web/service-worker.js` | http | public | ["GET"] |  | `web` | `WebManifest.service_worker` |  |
| `/odoo/offline` | http | public | ["GET"] |  | `web` | `WebManifest.offline` | Returns the offline page delivered by the service worker |
| `/scoped_app` | http | public | ["GET"] |  | `web` | `WebManifest.scoped_app` | Returns the app shortcut page to install the app given in parameters |
| `/scoped_app_icon_png` | http | public | ["GET"] |  | `web` | `WebManifest.scoped_app_icon_png` | Returns an app icon created with a fixed size in PNG. It is required for Safari PWAs |
| `/web/manifest.scoped_app_manifest` | http | public | ["GET"] |  | `web` | `WebManifest.scoped_app_manifest` | Returns a WebManifest dedicated to the scope of the given app. A custom scope and start url are set to make sure no other installed PWA can overlap the scope (e.g. /the system) |
| `/web_unsplash/attachment/add` | jsonrpc | user | ["POST"] |  | `web_unsplash` | `Web_Unsplash.save_unsplash_url` | unsplashurls = {     image_id1: {         url: image_url,         download_url: download_url,     },     image_id2: {         url: image_url,         download_url: download_url,     },     ..... } |
| `/web_unsplash/fetch_images` | jsonrpc | user |  |  | `web_unsplash` | `Web_Unsplash.fetch_unsplash_images` |  |
| `/web_unsplash/get_app_id` | jsonrpc | public |  |  | `web_unsplash` | `Web_Unsplash.get_unsplash_app_id` |  |
| `/web_unsplash/save_unsplash` | jsonrpc | user |  |  | `web_unsplash` | `Web_Unsplash.save_unsplash` |  |
| `/website/fetch_dashboard_data` | jsonrpc | user |  |  | `website` | `WebsiteBackend.fetch_dashboard_data` |  |
| `/website/iframefallback` | http | user |  | True | `website` | `WebsiteBackend.get_iframe_fallback` |  |
| `/website/check_new_content_access_rights` | jsonrpc | user |  |  | `website` | `WebsiteBackend.check_create_access_rights` | TODO: In master, remove this route and method and find a better way to do this. This route is only here to ensure that the "New Content" modal displays the correct elements for each user, and there might be a way to do it with the framework rather than having a dedicated controller route. (maybe by  |
| `/website/track_installing_modules` | jsonrpc | user |  |  | `website` | `WebsiteBackend.website_track_installing_modules` | During the website configuration, this route allows to track the website features being installed and their dependencies in order to show the progress between installed and yet to install features. |
| `/web/assets/<int:website_id>/<unique>/<string:filename>` | http | public |  |  | `website` | `WebsiteBinary.content_assets_website` |  |
| `/website/form` | http | public | ["POST"] |  | `website` | `WebsiteForm.website_form_empty` |  |
| `/website/form/<string:model_name>` | http | public | ["POST"] | True | `website` | `WebsiteForm.website_form` |  |
| `/` | http | public |  | True | `website` | `Website.index` | The goal of this controller is to make sure we don't serve a 404 as the website homepage. As this is the website entry point, serving a 404 is terrible. There is multiple fallback mechanism to prevent that: - If homepage URL is set (empty by default), serve the website.page matching it - If homepage |
| `/website/force/<int:website_id>` | http | user |  | True | `website` | `Website.website_force` | To switch from a website to another, we need to force the website in session, AFTER landing on that website domain (if set) as this will be a different session. |
| `/@/` | http | public |  | True | `website` | `Website.client_action_redirect` | Redirect internal users to the backend preview of the requested path URL (client action iframe). Non internal users will be redirected to the regular frontend version of that URL. |
| `/@/<path:path>` | http | public |  | True | `website` | `Website.client_action_redirect` | Redirect internal users to the backend preview of the requested path URL (client action iframe). Non internal users will be redirected to the regular frontend version of that URL. |
| `/website/get_languages` | jsonrpc | user |  | True | `website` | `Website.website_languages` |  |
| `/website/get_translated_elements` | jsonrpc | user |  |  | `website` | `Website.translated_elements` |  |
| `/website/lang/<lang>` | http | public |  | True | `website` | `Website.change_lang` | :param lang: supposed to be value of `url_code` field |
| `/website/country_infos/<model("res.country"):country>` | jsonrpc | public | ["POST"] | True | `website` | `Website.country_infos` |  |
| `/robots.txt` | http | public |  | True | `website` | `Website.robots` |  |
| `/sitemap.xml` | http | public |  | True | `website` | `Website.sitemap_xml_index` |  |
| `/favicon.ico` | http | public |  | True | `website` | `Website.favicon` |  |
| `/website/info` | http | public |  | True | `website` | `Website.website_info` |  |
| `/website/configurator` | http | user |  | True | `website` | `Website.website_configurator` |  |
| `/website/configurator/<int:step>` | http | user |  | True | `website` | `Website.website_configurator` |  |
| `/website/social/<string:social>` | http | public |  | True | `website` | `Website.social` |  |
| `/website/get_suggested_links` | jsonrpc | user |  | True | `website` | `Website.get_suggested_link` |  |
| `/website/check_existing_link` | jsonrpc | user |  | True | `website` | `Website.check_existing_link` |  |
| `/website/save_session_layout_mode` | jsonrpc | public |  | True | `website` | `Website.save_session_layout_mode` |  |
| `/website/snippet/filters` | jsonrpc | public |  | True | `website` | `Website.get_dynamic_filter` |  |
| `/website/snippet/options_filters` | jsonrpc | user |  | True | `website` | `Website.get_dynamic_snippet_filters` |  |
| `/website/snippet/filter_templates` | jsonrpc | public |  | True | `website` | `Website.get_dynamic_snippet_templates` |  |
| `/website/get_current_currency` | jsonrpc | public |  | True | `website` | `Website.get_current_currency` |  |
| `/website/snippet/autocomplete` | jsonrpc | public |  | True | `website` | `Website.autocomplete` | Returns list of results according to the term and options  :param str search_type: indicates what to search within, 'all' matches all available types :param str term: search term written by the user :param str order: :param int limit: number of results to consider, defaults to 5 :param int max_nb_ch |
| `/pages` | http | public |  | True | `website` | `Website.pages_list` |  |
| `/pages/page/<int:page>` | http | public |  | True | `website` | `Website.pages_list` |  |
| `/website/search` | http | public |  | True | `website` | `Website.hybrid_list` |  |
| `/website/search/page/<int:page>` | http | public |  | True | `website` | `Website.hybrid_list` |  |
| `/website/search/<string:search_type>` | http | public |  | True | `website` | `Website.hybrid_list` |  |
| `/website/search/<string:search_type>/page/<int:page>` | http | public |  | True | `website` | `Website.hybrid_list` |  |
| `/website/add` | http | user | ["POST"] | True | `website` | `Website.pagenew` |  |
| `/website/add/<path:path>` | http | user | ["POST"] | True | `website` | `Website.pagenew` |  |
| `/website/get_new_page_templates` | jsonrpc | user |  | True | `website` | `Website.get_new_page_templates` |  |
| `/website/save_xml` | jsonrpc | user |  | True | `website` | `Website.save_xml` |  |
| `/website/get_switchable_related_views` | jsonrpc | user |  | True | `website` | `Website.get_switchable_related_views` |  |
| `/website/reset_template` | jsonrpc | user | ["POST"] |  | `website` | `Website.reset_template` | This method will try to reset a broken view. Given the mode, the view can either be: - Soft reset: restore to previous architecture. - Hard reset: it will read the original `arch` from the XML file if the view comes from an XML file (arch_fs). |
| `/website/seo_suggest` | jsonrpc | user |  | True | `website` | `Website.seo_suggest` | Suggests search keywords based on a given input using Google's autocomplete API.  This method takes in a `keywords` string and an optional `lang` parameter that defines the language and geographical region for tailoring search suggestions. It sends a request to Google's autocomplete service and retu |
| `/website/get_alt_images` | jsonrpc | user |  | True | `website` | `Website.get_alt_images` |  |
| `/website/update_alt_images` | jsonrpc | user |  | True | `website` | `Website.update_alt_images` |  |
| `/website/update_broken_links` | jsonrpc | user |  | True | `website` | `Website.update_broken_links` |  |
| `/website/get_seo_data` | jsonrpc | user |  | True | `website` | `Website.get_seo_data` |  |
| `/website/check_can_modify_any` | jsonrpc | user |  | True | `website` | `Website.check_can_modify_any` |  |
| `/google<string(length=16):key>.html` | http | public |  | True | `website` | `Website.google_console_search` |  |
| `/website/google_maps_api_key` | jsonrpc | public |  | True | `website` | `Website.google_maps_api_key` |  |
| `/website/google_font_metadata` | jsonrpc | user |  | True | `website` | `Website.google_font_metadata` | Avoid CORS by caching google fonts metadata on server |
| `/website/theme_customize_data_get` | jsonrpc | user |  | True | `website` | `Website.theme_customize_data_get` |  |
| `/website/theme_customize_data` | jsonrpc | user |  | True | `website` | `Website.theme_customize_data` | Enables and/or disables views/assets according to list of keys.  :param is_view_data: True = "ir.ui.view", False = "ir.asset" :param enable: list of views/assets keys to enable :param disable: list of views/assets keys to disable :param reset_view_arch: restore the default template after disabling |
| `/website/theme_customize_bundle_reload` | jsonrpc | user |  | True | `website` | `Website.theme_customize_bundle_reload` | Reloads asset bundles and returns their unique URLs. |
| `/website/update_footer_template` | jsonrpc | user |  | True | `website` | `Website.update_footer_template` | Enables the footer template and its corresponding copyright template on template change. The goal is to ensure that the content width of the copyright aligns with the footer. |
| `/website/theme_upload_font` | jsonrpc | user |  | True | `website` | `Website.theme_upload_font` | Uploads font binary data and returns metadata about accessing individual fonts. :param name: name of the uploaded file :param data: binary content of the uploaded file :return: list of dict describing each contained font with:     - name     - mimetype     - attachment id     - attachment URL |
| `/website/action/<path_or_xml_id_or_id>` | http | public |  | True | `website` | `Website.actions_server` |  |
| `/website/action/<path_or_xml_id_or_id>/<path:path>` | http | public |  | True | `website` | `Website.actions_server` |  |
| `/website/get_assets_editor_resources` | jsonrpc | user |  | True | `website` | `Website.get_assets_editor_resources` | Transmit the resources the assets editor needs to work.  Params:     key (str): the key of the view the resources are related to      get_views (bool, default=True):         True if the views must be fetched      get_scss (bool, default=True):         True if the style must be fetched      get_js (b |
| `/website/field/translation/update` | jsonrpc | user |  | True | `website` | `Website.update_field_translation` |  |
| `/website/image` | http | public |  |  | `website` | `WebsiteBinary.website_content_image` |  |
| `/website/image/<xmlid>` | http | public |  |  | `website` | `WebsiteBinary.website_content_image` |  |
| `/website/image/<xmlid>/<int:width>x<int:height>` | http | public |  |  | `website` | `WebsiteBinary.website_content_image` |  |
| `/website/image/<xmlid>/<field>` | http | public |  |  | `website` | `WebsiteBinary.website_content_image` |  |
| `/website/image/<xmlid>/<field>/<int:width>x<int:height>` | http | public |  |  | `website` | `WebsiteBinary.website_content_image` |  |
| `/website/image/<model>/<id>/<field>` | http | public |  |  | `website` | `WebsiteBinary.website_content_image` |  |
| `/website/image/<model>/<id>/<field>/<int:width>x<int:height>` | http | public |  |  | `website` | `WebsiteBinary.website_content_image` |  |
| `/model/<string:page_name_slugified>` | http | public |  | True | `website` | `ModelPageController.generic_model` |  |
| `/model/<string:page_name_slugified>/page/<int:page_number>` | http | public |  | True | `website` | `ModelPageController.generic_model` |  |
| `/model/<string:page_name_slugified>/<string:record_slug>` | http | public |  | True | `website` | `ModelPageController.generic_model` |  |
| `/blog` | http | public |  | True | `website_blog` | `WebsiteBlog.blog` |  |
| `/blog/page/<int:page>` | http | public |  | True | `website_blog` | `WebsiteBlog.blog` |  |
| `/blog/tag/<string:tag>` | http | public |  | True | `website_blog` | `WebsiteBlog.blog` |  |
| `/blog/tag/<string:tag>/page/<int:page>` | http | public |  | True | `website_blog` | `WebsiteBlog.blog` |  |
| `/blog/<model("blog.blog"):blog>` | http | public |  | True | `website_blog` | `WebsiteBlog.blog` |  |
| `/blog/<model("blog.blog"):blog>/page/<int:page>` | http | public |  | True | `website_blog` | `WebsiteBlog.blog` |  |
| `/blog/<model("blog.blog"):blog>/tag/<string:tag>` | http | public |  | True | `website_blog` | `WebsiteBlog.blog` |  |
| `/blog/<model("blog.blog"):blog>/tag/<string:tag>/page/<int:page>` | http | public |  | True | `website_blog` | `WebsiteBlog.blog` |  |
| `/blog/<model("blog.blog"):blog>/feed` | http | public |  | True | `website_blog` | `WebsiteBlog.blog_feed` |  |
| `/blog/<model("blog.blog"):blog>/post/<model("blog.post"):blog_post>` | http | public |  | True | `website_blog` | `WebsiteBlog.old_blog_post` |  |
| `/blog/<model("blog.blog"):blog>/<model("blog.post", "[('blog_id','=',blog.id)]"):blog_post>` | http | public |  | True | `website_blog` | `WebsiteBlog.blog_post` | Prepare all values to display the blog.  :return dict values: values for the templates, containing   - 'blog_post': browse of the current post  - 'blog': browse of the current blog  - 'blogs': list of browse records of blogs  - 'tag': current tag, if tag_id in parameters  - 'tags': all tags, for tag |
| `/my/leads` | http | user |  | True | `website_crm_partner_assign` | `WebsiteAccount.portal_my_leads` |  |
| `/my/leads/page/<int:page>` | http | user |  | True | `website_crm_partner_assign` | `WebsiteAccount.portal_my_leads` |  |
| `/my/opportunities` | http | user |  | True | `website_crm_partner_assign` | `WebsiteAccount.portal_my_opportunities` |  |
| `/my/opportunities/page/<int:page>` | http | user |  | True | `website_crm_partner_assign` | `WebsiteAccount.portal_my_opportunities` |  |
| `/my/lead/<model('crm.lead', "[('type','=', 'lead')]"):lead>` | http | user |  | True | `website_crm_partner_assign` | `WebsiteAccount.portal_my_lead` |  |
| `/my/opportunity/<model('crm.lead', "[('type','=', 'opportunity')]"):opp>` | http | user |  | True | `website_crm_partner_assign` | `WebsiteAccount.portal_my_opportunity` |  |
| `/partners` | http | public |  | True | `website_crm_partner_assign` | `WebsiteCrmPartnerAssign.partners` |  |
| `/partners/page/<int:page>` | http | public |  | True | `website_crm_partner_assign` | `WebsiteCrmPartnerAssign.partners` |  |
| `/partners/grade/<model("res.partner.grade"):grade>` | http | public |  | True | `website_crm_partner_assign` | `WebsiteCrmPartnerAssign.partners` |  |
| `/partners/grade/<model("res.partner.grade"):grade>/page/<int:page>` | http | public |  | True | `website_crm_partner_assign` | `WebsiteCrmPartnerAssign.partners` |  |
| `/partners/country/<model("res.country"):country>` | http | public |  | True | `website_crm_partner_assign` | `WebsiteCrmPartnerAssign.partners` |  |
| `/partners/country/<model("res.country"):country>/page/<int:page>` | http | public |  | True | `website_crm_partner_assign` | `WebsiteCrmPartnerAssign.partners` |  |
| `/partners/grade/<model("res.partner.grade"):grade>/country/<model("res.country"):country>` | http | public |  | True | `website_crm_partner_assign` | `WebsiteCrmPartnerAssign.partners` |  |
| `/partners/grade/<model("res.partner.grade"):grade>/country/<model("res.country"):country>/page/<int:page>` | http | public |  | True | `website_crm_partner_assign` | `WebsiteCrmPartnerAssign.partners` |  |
| `/customers` | http | public |  | True | `website_customer` | `WebsiteCustomer.customers` |  |
| `/customers/page/<int:page>` | http | public |  | True | `website_customer` | `WebsiteCustomer.customers` |  |
| `/customers/country/<model("res.country"):country>` | http | public |  | True | `website_customer` | `WebsiteCustomer.customers` |  |
| `/customers/country/<model("res.country"):country>/page/<int:page>` | http | public |  | True | `website_customer` | `WebsiteCustomer.customers` |  |
| `/customers/industry/<model("res.partner.industry"):industry>` | http | public |  | True | `website_customer` | `WebsiteCustomer.customers` |  |
| `/customers/industry/<model("res.partner.industry"):industry>/page/<int:page>` | http | public |  | True | `website_customer` | `WebsiteCustomer.customers` |  |
| `/customers/industry/<model("res.partner.industry"):industry>/country/<model("res.country"):country>` | http | public |  | True | `website_customer` | `WebsiteCustomer.customers` |  |
| `/customers/industry/<model("res.partner.industry"):industry>/country/<model("res.country"):country>/page/<int:page>` | http | public |  | True | `website_customer` | `WebsiteCustomer.customers` |  |
| `/customers/<partner_id>` | http | public |  | True | `website_customer` | `WebsiteCustomer.customers_detail` |  |
| `/event/<model("event.event"):event>/community` | http | public |  | True | `website_event` | `EventCommunityController.community` | This skeleton route will be overriden in website_event_track_quiz. |
| `expression` | http | public |  | True | `website_event` | `WebsiteEventController.events` |  |
| `/event/<model("event.event"):event>/page/<path:page>` | http | public |  | True | `website_event` | `WebsiteEventController.event_page` |  |
| `/event/<model("event.event"):event>` | http | public |  | True | `website_event` | `WebsiteEventController.event` |  |
| `/event/<model("event.event"):event>/register` | http | public |  | True | `website_event` | `WebsiteEventController.event_register` |  |
| `/event/<model("event.event"):event>/registration/slot/<int:slot_id>/tickets` | jsonrpc | public | ["POST"] | True | `website_event` | `WebsiteEventController.registration_tickets` | After slot selection, render ticket selection modal. To restrict the selectable number of tickets, give the slot seats available and each slot tickets seats available to the template. |
| `/event/<model("event.event"):event>/registration/new` | jsonrpc | public | ["POST"] | True | `website_event` | `WebsiteEventController.registration_new` | After (slot and) tickets selection, render attendee(s) registration form. Slot and tickets availability check already performed in the template. |
| `/event/<model("event.event"):event>/registration/confirm` | http | public | ["POST"] | True | `website_event` | `WebsiteEventController.registration_confirm` | Check before creating and finalize the creation of the registrations that we have enough seats for all selected tickets. If we don't, the user is instead redirected to page to register with a formatted error message. |
| `/event/<model("event.event"):event>/registration/success` | http | public | ["GET"] | True | `website_event` | `WebsiteEventController.event_registration_success` |  |
| `/event/<model("event.event"):event>/booth` | http | public |  | True | `website_event_booth` | `WebsiteEventBoothController.event_booth_main` |  |
| `/event/<model("event.event"):event>/booth/register` | http | public | ["POST"] | True | `website_event_booth` | `WebsiteEventBoothController.event_booth_register` |  |
| `/event/<model("event.event"):event>/booth/register_form` | http | public | ["GET"] | True | `website_event_booth` | `WebsiteEventBoothController.event_booth_contact_form` |  |
| `/event/<model("event.event"):event>/booth/confirm` | http | public | ["POST"] | True | `website_event_booth` | `WebsiteEventBoothController.event_booth_registration_confirm` |  |
| `/event/booth/check_availability` | jsonrpc | public | ["POST"] |  | `website_event_booth` | `WebsiteEventBoothController.check_booths_availability` |  |
| `/event/booth_category/get_available_booths` | jsonrpc | public |  |  | `website_event_booth` | `WebsiteEventBoothController.get_booth_category_available_booths` |  |
| `/event/<model("event.event"):event>/exhibitors` | http | public | ["GET", "POST"] | True | `website_event_exhibitor` | `ExhibitorController.event_exhibitors` |  |
| `/event/<model("event.event"):event>/exhibitor` | http | public | ["GET", "POST"] | True | `website_event_exhibitor` | `ExhibitorController.event_exhibitors` |  |
| `/event/<model("event.event", "[('exhibitor_menu', '=', True)]"):event>/exhibitor/<model("event.sponsor", "[('event_id', '=', event.id)]"):sponsor>` | http | public |  | True | `website_event_exhibitor` | `ExhibitorController.event_exhibitor` |  |
| `/event_sponsor/<int:sponsor_id>/read` | jsonrpc | public |  | True | `website_event_exhibitor` | `ExhibitorController.event_sponsor_read` | Marshmalling data for "event not started / sponsor not available" modal |
| `/event/<model("event.event"):event>/track` | http | public |  | True | `website_event_track` | `EventTrackController.event_tracks` | Main route  :param event: event whose tracks are about to be displayed; :param tag: deprecated: search for a specific tag :param searches: frontend search dict, containing    * 'search': search string;   * 'tags': list of tag IDs for filtering; |
| `/event/<model("event.event"):event>/track/tag/<model("event.track.tag"):tag>` | http | public |  | True | `website_event_track` | `EventTrackController.event_tracks` | Main route  :param event: event whose tracks are about to be displayed; :param tag: deprecated: search for a specific tag :param searches: frontend search dict, containing    * 'search': search string;   * 'tags': list of tag IDs for filtering; |
| `/event/<model("event.event"):event>/agenda` | http | public |  | True | `website_event_track` | `EventTrackController.event_agenda` |  |
| `/event/<model("event.event", "[('website_track', '=', True)]"):event>/track/<model("event.track", "[('event_id', '=', event.id)]"):track>` | http | public |  | True | `website_event_track` | `EventTrackController.event_track_page` |  |
| `/event/track/toggle_reminder` | jsonrpc | public |  | True | `website_event_track` | `EventTrackController.track_reminder_toggle` | Set a reminder a track for current visitor. Track visitor is created or updated if it already exists. Exception made if un-favoriting and no track_visitor record found (should not happen unless manually done).  :param boolean set_reminder_on:   If True, set as a favorite, otherwise un-favorite track |
| `/event/track/send_email_reminder` | jsonrpc | public |  | True | `website_event_track` | `EventTrackController.send_email_reminder` | Send email, to email_to if the user is public otherwise to the user email address, with tracks' reminders for external calendars. |
| `/event/<model("event.event"):event>/track_proposal` | http | public |  | True | `website_event_track` | `EventTrackController.event_track_proposal` |  |
| `/event/<model("event.event"):event>/track_proposal/post` | http | public | ["POST"] | True | `website_event_track` | `EventTrackController.event_track_proposal_post` |  |
| `/event/track_tag/search_read` | jsonrpc | public |  | True | `website_event_track` | `EventTrackController.website_event_track_fetch_tags` |  |
| `/event/<model("event.event"):event>/track/<model("event.track"):track>/ics` | http | public |  | True | `website_event_track` | `EventTrackController.event_track_ics_file` |  |
| `/event/manifest.webmanifest` | http | public | ["GET"] | True | `website_event_track` | `TrackManifest.webmanifest` | Returns a WebManifest describing the metadata associated with a web application. Using this metadata, user agents can provide developers with means to create user  experiences that are more comparable to that of a native application. |
| `/event/service-worker.js` | http | public | ["GET"] | True | `website_event_track` | `TrackManifest.service_worker` | Returns a ServiceWorker javascript file scoped for website_event |
| `/event/offline` | http | public | ["GET"] | True | `website_event_track` | `TrackManifest.offline` | Returns the offline page used by the 'website_event' PWA |
| `/event_track/get_track_suggestion` | jsonrpc | public |  | True | `website_event_track_live` | `EventTrackLiveController.get_next_track_suggestion` |  |
| `/event/<model("event.event"):event>/community/leaderboard/results` | http | public |  | True | `website_event_track_quiz` | `WebsiteEventTrackQuizCommunityController.leaderboard` |  |
| `/event/<model("event.event"):event>/community/leaderboard/results/page/<int:page>` | http | public |  | True | `website_event_track_quiz` | `WebsiteEventTrackQuizCommunityController.leaderboard` |  |
| `/event/<model("event.event"):event>/community/leaderboard` | http | public |  | True | `website_event_track_quiz` | `WebsiteEventTrackQuizCommunityController.community_leaderboard` |  |
| `/event_track/quiz/submit` | jsonrpc | public |  | True | `website_event_track_quiz` | `WebsiteEventTrackQuiz.event_track_quiz_submit` |  |
| `/event_track/quiz/reset` | jsonrpc | public |  | True | `website_event_track_quiz` | `WebsiteEventTrackQuiz.quiz_reset` |  |
| `/forum/user/<int:user_id>` | http | public |  | True | `website_forum` | `WebsiteForumLegacy.view_user_forum_profile` |  |
| `/forum` | http | public |  | True | `website_forum` | `WebsiteForum.forum` |  |
| `/forum/all` | http | public |  | True | `website_forum` | `WebsiteForum.questions` |  |
| `/forum/all/page/<int:page>` | http | public |  | True | `website_forum` | `WebsiteForum.questions` |  |
| `/forum/<model("forum.forum"):forum>` | http | public |  | True | `website_forum` | `WebsiteForum.questions` |  |
| `/forum/<model("forum.forum"):forum>/page/<int:page>` | http | public |  | True | `website_forum` | `WebsiteForum.questions` |  |
| `/forum/<model("forum.forum"):forum>/tag/<model("forum.tag"):tag>/questions` | http | public |  | True | `website_forum` | `WebsiteForum.questions` |  |
| `/forum/<model("forum.forum"):forum>/tag/<model("forum.tag"):tag>/questions/page/<int:page>` | http | public |  | True | `website_forum` | `WebsiteForum.questions` |  |
| `/forum/<model("forum.forum"):forum>/faq` | http | public |  | True | `website_forum` | `WebsiteForum.forum_faq` |  |
| `/forum/<model("forum.forum"):forum>/faq/karma` | http | public |  | True | `website_forum` | `WebsiteForum.forum_faq_karma` |  |
| `/forum/get_tags` | http | public | ["GET"] | True | `website_forum` | `WebsiteForum.tag_read` |  |
| `/forum/<model("forum.forum"):forum>/tag` | http | public |  | True | `website_forum` | `WebsiteForum.tags` | Render a list of tags matching filters and search parameters.  :param forum: Forum :param string tag_char: Only tags starting with a single character `tag_char` :param filters: One of 'all'\|'followed'\|'most_used'\|'unused'.   Can be combined with `search` and `tag_char`. :param string search: Sear |
| `/forum/<model("forum.forum"):forum>/tag/<string:tag_char>` | http | public |  | True | `website_forum` | `WebsiteForum.tags` | Render a list of tags matching filters and search parameters.  :param forum: Forum :param string tag_char: Only tags starting with a single character `tag_char` :param filters: One of 'all'\|'followed'\|'most_used'\|'unused'.   Can be combined with `search` and `tag_char`. :param string search: Sear |
| `/forum/get_url_title` | jsonrpc | user | ["POST"] | True | `website_forum` | `WebsiteForum.get_url_title` |  |
| `/forum/<model("forum.forum"):forum>/question/<model("forum.post", "[('forum_id','=',forum.id),('parent_id','=',False),('can_view', '=', True)]"):question>` | http | public |  | True | `website_forum` | `WebsiteForum.old_question` |  |
| `/forum/<model("forum.forum"):forum>/<model("forum.post"):question>` | http | public |  | True | `website_forum` | `WebsiteForum.question` |  |
| `/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/toggle_favourite` | jsonrpc | user | ["POST"] | True | `website_forum` | `WebsiteForum.question_toggle_favorite` |  |
| `/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/ask_for_close` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.question_ask_for_close` |  |
| `/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/edit_answer` | http | user |  | True | `website_forum` | `WebsiteForum.question_edit_answer` |  |
| `/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/close` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.question_close` |  |
| `/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/reopen` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.question_reopen` |  |
| `/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/delete` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.question_delete` |  |
| `/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/undelete` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.question_undelete` |  |
| `/forum/<model("forum.forum"):forum>/ask` | http | user |  | True | `website_forum` | `WebsiteForum.forum_post` |  |
| `/forum/<model("forum.forum"):forum>/new` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.post_create` |  |
| `/forum/<model("forum.forum"):forum>/<model("forum.post"):post_parent>/reply` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.post_create` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/comment` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.post_comment` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/toggle_correct` | jsonrpc | user |  | True | `website_forum` | `WebsiteForum.post_toggle_correct` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/delete` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.post_delete` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/edit` | http | user |  | True | `website_forum` | `WebsiteForum.post_edit` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/save` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.post_save` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/upvote` | jsonrpc | user |  | True | `website_forum` | `WebsiteForum.post_upvote` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/downvote` | jsonrpc | user |  | True | `website_forum` | `WebsiteForum.post_downvote` |  |
| `/forum/<model("forum.forum"):forum>/validation_queue` | http | user |  | True | `website_forum` | `WebsiteForum.validation_queue` |  |
| `/forum/<model("forum.forum"):forum>/flagged_queue` | http | user |  | True | `website_forum` | `WebsiteForum.flagged_queue` |  |
| `/forum/<model("forum.forum"):forum>/offensive_posts` | http | user |  | True | `website_forum` | `WebsiteForum.offensive_posts` |  |
| `/forum/<model("forum.forum"):forum>/closed_posts` | http | user |  | True | `website_forum` | `WebsiteForum.closed_posts` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/validate` | http | user |  | True | `website_forum` | `WebsiteForum.post_accept` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/refuse` | http | user |  | True | `website_forum` | `WebsiteForum.post_refuse` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/flag` | jsonrpc | user |  | True | `website_forum` | `WebsiteForum.post_flag` |  |
| `/forum/<model("forum.post"):post>/ask_for_mark_as_offensive` | jsonrpc | user |  | True | `website_forum` | `WebsiteForum.post_json_ask_for_mark_as_offensive` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/ask_for_mark_as_offensive` | http | user | ["GET"] | True | `website_forum` | `WebsiteForum.post_http_ask_for_mark_as_offensive` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/mark_as_offensive` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.post_mark_as_offensive` |  |
| `/forum/<model("forum.forum"):forum>/partner/<int:partner_id>` | http | public |  | True | `website_forum` | `WebsiteForum.open_partner` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/comment/<model("mail.message"):comment>/convert_to_answer` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.convert_comment_to_answer` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/convert_to_comment` | http | user | ["POST"] | True | `website_forum` | `WebsiteForum.convert_answer_to_comment` |  |
| `/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/comment/<model("mail.message"):comment>/delete` | jsonrpc | user |  | True | `website_forum` | `WebsiteForum.delete_comment` |  |
| `/google_map` | http | public |  | True | `website_google_map` | `GoogleMap.google_map` |  |
| `/jobs` | http | public |  | True | `website_hr_recruitment` | `WebsiteHrRecruitment.jobs` | This method is returning the job page. It's filtering the jobs by the given parameters and compute the display values for the filters by contaminating the jobs with the other filters. |
| `/jobs/page/<int:page>` | http | public |  | True | `website_hr_recruitment` | `WebsiteHrRecruitment.jobs` | This method is returning the job page. It's filtering the jobs by the given parameters and compute the display values for the filters by contaminating the jobs with the other filters. |
| `/jobs/add` | jsonrpc | user |  | True | `website_hr_recruitment` | `WebsiteHrRecruitment.jobs_add` |  |
| `/jobs/detail/<model("hr.job"):job>` | http | public |  | True | `website_hr_recruitment` | `WebsiteHrRecruitment.jobs_detail` |  |
| `/jobs/<model("hr.job"):job>` | http | public |  | True | `website_hr_recruitment` | `WebsiteHrRecruitment.job` |  |
| `/jobs/apply/<model("hr.job"):job>` | http | public |  | True | `website_hr_recruitment` | `WebsiteHrRecruitment.jobs_apply` |  |
| `/website_hr_recruitment/check_recent_application` | jsonrpc | public |  | True | `website_hr_recruitment` | `WebsiteHrRecruitment.check_recent_application` |  |
| `/website_links/new` | jsonrpc | user | ["POST"] |  | `website_links` | `WebsiteUrl.create_shorten_url` |  |
| `/r` | http | user |  | True | `website_links` | `WebsiteUrl.shorten_url` |  |
| `/website_links/add_code` | jsonrpc | user |  |  | `website_links` | `WebsiteUrl.add_code` |  |
| `/website_links/recent_links` | jsonrpc | user |  |  | `website_links` | `WebsiteUrl.recent_links` |  |
| `/r/<string:code>+` | http | user |  | True | `website_links` | `WebsiteUrl.statistics_shorten_url` |  |
| `/chatbot/<model("chatbot.script"):chatbot_script>/test` | http | user |  | True | `website_livechat` | `WebsiteLivechatChatbotScriptController.chatbot_test_script` | Custom route allowing to test a chatbot script. As we don't have a im_livechat.channel linked to it, we pre-emptively create a discuss.channel that will hold the conversation between the bot and the user testing the script. |
| `/website_mail/follow` | jsonrpc | public |  | True | `website_mail` | `WebsiteMail.website_message_subscribe` |  |
| `/website_mail/is_follower` | jsonrpc | public |  | True | `website_mail` | `WebsiteMail.is_follower` | Given a list of `models` containing a list of res_ids, return the res_ids for which the user is follower and some practical info.  :param records: dict of models containing record IDS, eg: {         'res.model': [1, 2, 3..],         'res.model2': [1, 2, 3..],         ..     }  :returns: [         {' |
| `/group/is_member` | jsonrpc | public |  | True | `website_mail_group` | `WebsiteMailGroup.group_is_member` | Return the email of the member if found, otherwise None. |
| `/website_mass_mailing/is_subscriber` | jsonrpc | public |  | True | `website_mass_mailing` | `MassMailController.is_subscriber` |  |
| `/website_mass_mailing/subscribe` | jsonrpc | public |  | True | `website_mass_mailing` | `MassMailController.subscribe` |  |
| `/partners/<partner_id>` | http | public |  | True | `website_partner` | `WebsitePartnerPage.partners_detail` |  |
| `/donation/pay` | http | public | ["GET", "POST"] | True | `website_payment` | `PaymentPortal.donation_pay` | Behaves like PaymentPortal.payment_pay but for donation  :param dict kwargs: As the parameters of in payment_pay, with the additional:     - str donation_options: The options settled in the donation snippet     - str donation_descriptions: The descriptions for all prefilled amounts :return: The rend |
| `/donation/transaction/<minimum_amount>` | jsonrpc | public |  | True | `website_payment` | `PaymentPortal.donation_transaction` |  |
| `/website_payment/snippet/supported_payment_methods` | http | public | ["GET"] | True | `website_payment` | `PaymentPortal.get_supported_payment_methods` | Retrieve the payment methods linked to payment providers published on the current website.  If a payment method is a primary payment method, its brands are returned instead.  Note: The provider must be linked to the same company as the website. This differs from the usual payment method selection, w |
| `/profile/avatar/<int:user_id>` | http | public |  | True | `website_profile` | `WebsiteProfile.get_user_profile_avatar` |  |
| `/profile/user/<int:user_id>` | http | public |  | True | `website_profile` | `WebsiteProfile.view_user_profile` |  |
| `/profile/user/save` | jsonrpc | user | ["POST"] | True | `website_profile` | `WebsiteProfile.save_edited_profile` |  |
| `/profile/ranks_badges` | http | public |  | True | `website_profile` | `WebsiteProfile.view_ranks_badges` |  |
| `/profile/users` | http | public |  | True | `website_profile` | `WebsiteProfile.view_all_users_page` |  |
| `/profile/users/page/<int:page>` | http | public |  | True | `website_profile` | `WebsiteProfile.view_all_users_page` |  |
| `/profile/send_validation_email` | jsonrpc | user |  | True | `website_profile` | `WebsiteProfile.send_validation_email` |  |
| `/profile/validate_email` | http | public |  | True | `website_profile` | `WebsiteProfile.validate_email` |  |
| `/profile/validate_email/close` | jsonrpc | public |  | True | `website_profile` | `WebsiteProfile.validate_email_done` |  |
| `/shop/cart` | http | public |  | True | `website_sale` | `Cart.cart` | Display the cart page.  This route is responsible for the main cart management and abandoned cart revival logic.  :param str id: The abandoned cart's id. :param str access_token: The abandoned cart's access token. :param str revive_method: The revival method for abandoned carts. Can be 'merge' or 's |
| `/shop/cart/add` | jsonrpc | public | ["POST"] | True | `website_sale` | `Cart.add_to_cart` | Adds a product to the shopping cart.  :param int product_template_id: The product to add to cart, as a     `product.template` id. :param int product_id: The product to add to cart, as a     `product.product` id. :param int quantity: The quantity to add to the cart. :param list[dict] product_custom_a |
| `/shop/cart/quick_add` | jsonrpc | user | ["POST"] | True | `website_sale` | `Cart.quick_add` |  |
| `/shop/cart/update` | jsonrpc | public | ["POST"] | True | `website_sale` | `Cart.update_cart` | Update the quantity of a specific line of the current cart.  :param int line_id: line to update, as a `sale.order.line` id. :param float quantity: new line quantity.     0 or negative numbers will only delete the line, the ecommerce     doesn't work with negative numbers. :param int\|None product_id |
| `/shop/cart/quantity` | jsonrpc | public | ["POST"] | True | `website_sale` | `Cart.cart_quantity` |  |
| `/shop/cart/clear` | jsonrpc | public |  | True | `website_sale` | `Cart.clear_cart` |  |
| `/website_sale/combo_configurator/get_data` | jsonrpc | public |  | True | `website_sale` | `WebsiteSaleComboConfiguratorController.website_sale_combo_configurator_get_data` |  |
| `/website_sale/combo_configurator/get_price` | jsonrpc | public |  | True | `website_sale` | `WebsiteSaleComboConfiguratorController.website_sale_combo_configurator_get_price` |  |
| `/shop/delivery_methods` | jsonrpc | public |  | True | `website_sale` | `Delivery.shop_delivery_methods` | Fetch available delivery methods and render them in the delivery form.  :return: The rendered delivery form. :rtype: str |
| `/shop/set_delivery_method` | jsonrpc | public |  | True | `website_sale` | `Delivery.shop_set_delivery_method` | Set the delivery method on the current order and return the order summary values.  If the delivery method is already set, the order summary values are returned immediately.  :param str dm_id: The delivery method to set, as a `delivery.carrier` id. :param dict kwargs: The keyword arguments forwarded  |
| `/shop/get_delivery_rate` | jsonrpc | public | ["POST"] | True | `website_sale` | `Delivery.shop_get_delivery_rate` | Return the delivery rate data for the given delivery method.  :param str dm_id: The delivery method whose rate to get, as a `delivery.carrier` id. :return: The delivery rate data. :rtype: dict |
| `/website_sale/set_pickup_location` | jsonrpc | public |  | True | `website_sale` | `Delivery.website_sale_set_pickup_location` | Fetch the order from the request and set the pickup location on the current order.  :param str pickup_location_data: The JSON-formatted pickup location address. :return: None |
| `/website_sale/get_pickup_locations` | jsonrpc | public |  | True | `website_sale` | `Delivery.website_sale_get_pickup_locations` | Fetch the order from the request and return the pickup locations close to the zip code.  Determine the country based on GeoIP or fallback on the order's delivery address' country.  :param int zip_code: The zip code to look up to. :return: The close pickup locations data. :rtype: dict |
| `expression` | jsonrpc | public |  | True | `website_sale` | `Delivery.express_checkout_process_delivery_address` | Process the shipping address and return the available delivery methods.  Depending on whether the partner is registered and logged in, a new partner is created or we use an existing partner that matches the partial delivery address received.  :param dict partial_delivery_address: The delivery inform |
| `expression` | http | public |  | True | `website_sale` | `WebsiteSale.shop` |  |
| `expression` | http | public |  | True | `website_sale` | `WebsiteSale.product` |  |
| `/shop/<model("product.template"):product_template>/document/<int:document_id>` | http | public |  | True | `website_sale` | `WebsiteSale.product_document` |  |
| `expression` | http | public |  | True | `website_sale` | `WebsiteSale.old_product` |  |
| `/shop/product/extra-media` | jsonrpc | user |  | True | `website_sale` | `WebsiteSale.add_product_media` | Handles adding both images and videos to product variants or templates, links all of them to product. :param type: [...] can be either image or video :raises NotFound : If the user is not allowed to access Attachment model |
| `/shop/product/clear-images` | jsonrpc | user |  | True | `website_sale` | `WebsiteSale.clear_product_images` | Unlinks all images from the product. |
| `/shop/product/resequence-image` | jsonrpc | user |  | True | `website_sale` | `WebsiteSale.resequence_product_image` | Move the product image in the given direction and update all images' sequence.  :param str image_res_model: The model of the image. It can be 'product.template',                             'product.product', or 'product.image'. :param str image_res_id: The record ID of the image to move. :param str |
| `/shop/product/is_add_to_cart_allowed` | jsonrpc | public |  | True | `website_sale` | `WebsiteSale.is_add_to_cart_allowed` |  |
| `/shop/change_pricelist/<model("product.pricelist"):pricelist>` | http | public |  | True | `website_sale` | `WebsiteSale.pricelist_change` |  |
| `/shop/pricelist` | http | public |  | True | `website_sale` | `WebsiteSale.pricelist` |  |
| `/shop/save_shop_layout_mode` | jsonrpc | public |  | True | `website_sale` | `WebsiteSale.save_shop_layout_mode` |  |
| `/shop/checkout` | http | public | ["GET"] | True | `website_sale` | `WebsiteSale.shop_checkout` | Display the checkout page.  :param str try_skip_step: Whether the user should immediately be redirected to the next step                           if no additional information (i.e., address or delivery method) is                           required on the checkout page. 'true' or 'false'. :param dic |
| `/shop/address` | http | public | ["GET"] | True | `website_sale` | `WebsiteSale.shop_address` | Display the address form.  A partner and/or an address type can be given through the query string params to specify which address to update or create, and its type.  :param str partner_id: The partner whose address to update with the address form, if any. :param str address_type: The type of the add |
| `/shop/address/submit` | http | public | ["POST"] | True | `website_sale` | `WebsiteSale.shop_address_submit` | Create or update an address.  If it succeeds, it returns the URL to redirect (client-side) to. If it fails (missing or invalid information), it highlights the problematic form input with the appropriate error message.  :param str partner_id: The partner whose address to update with the address form, |
| `expression` | jsonrpc | public | ["POST"] | True | `website_sale` | `WebsiteSale.process_express_checkout` | Records the partner information on the order when using express checkout flow.  Depending on whether the partner is registered and logged in, either creates a new partner or uses an existing one that matches all received data.  :param dict billing_address: Billing information sent by the express pay |
| `/shop/update_address` | jsonrpc | public |  | True | `website_sale` | `WebsiteSale.shop_update_address` |  |
| `/shop/extra_info` | http | public |  | True | `website_sale` | `WebsiteSale.extra_info` |  |
| `expression` | jsonrpc | public |  | True | `website_sale` | `WebsiteSale.express_checkout_shipping_address_compute_taxes` |  |
| `/shop/payment` | http | public |  | True | `website_sale` | `WebsiteSale.shop_payment` | Payment step. This page proposes several payment means based on available payment.provider. State at this point :   - a draft sales order with lines; otherwise, clean context / session and    back to the shop  - no transaction in context / session, or only a draft one, if the customer    did go to a |
| `/shop/payment/validate` | http | public |  | True | `website_sale` | `WebsiteSale.shop_payment_validate` | Method that should be called by the server when receiving an update for a transaction. State at this point :   - UDPATE ME |
| `/shop/confirmation` | http | public |  | True | `website_sale` | `WebsiteSale.shop_payment_confirmation` | End of checkout process controller. Confirmation is basically seing the status of a sale.order. State at this point :   - should not have any context / session info: clean them  - take a sale.order id, because we request a sale.order and are not    session dependant anymore |
| `/shop/print` | http | public |  | True | `website_sale` | `WebsiteSale.print_saleorder` |  |
| `/shop/config/product` | jsonrpc | user |  |  | `website_sale` | `WebsiteSale.change_product_config` |  |
| `/shop/config/attribute` | jsonrpc | user |  |  | `website_sale` | `WebsiteSale.change_attribute_config` |  |
| `/shop/config/website` | jsonrpc | user |  |  | `website_sale` | `WebsiteSale._change_website_config` |  |
| `/shop/config/category` | jsonrpc | user |  |  | `website_sale` | `WebsiteSale._change_category_config` |  |
| `/shop/products/recently_viewed_update` | jsonrpc | public |  | True | `website_sale` | `WebsiteSale.products_recently_viewed_update` |  |
| `/shop/products/recently_viewed_delete` | jsonrpc | public |  | True | `website_sale` | `WebsiteSale.products_recently_viewed_delete` |  |
| `/snippets/category/set_image` | jsonrpc | user |  |  | `website_sale` | `WebsiteSale.set_category_image` | Set the cover image on the category.  :param int category_id: ID of the category to set the cover image. :param int attachment_id: ID of the attachment containing the image data. :raise Forbidden: If the user does not have website editing access |
| `/shop/payment/transaction/<int:order_id>` | jsonrpc | public |  | True | `website_sale` | `PaymentPortal.shop_payment_transaction` | Create a draft transaction and return its processing values.  :param int order_id: The sales order to pay, as a `sale.order` id :param str access_token: The access token used to authenticate the request :param dict kwargs: Locally unused data passed to `_create_transaction` :return: The mandatory va |
| `/website_sale/should_show_product_configurator` | jsonrpc | public |  | True | `website_sale` | `WebsiteSaleProductConfiguratorController.website_sale_should_show_product_configurator` | Return whether the product configurator dialog should be shown.  :param int product_template_id: The product being checked, as a `product.template` id. :param list(int) ptav_ids: The combination of the product, as a list of     `product.template.attribute.value` ids. :param bool is_product_configure |
| `/website_sale/product_configurator/get_values` | jsonrpc | public |  | True | `website_sale` | `WebsiteSaleProductConfiguratorController.website_sale_product_configurator_get_values` |  |
| `/website_sale/product_configurator/create_product` | jsonrpc | public | ["POST"] | True | `website_sale` | `WebsiteSaleProductConfiguratorController.website_sale_product_configurator_create_product` |  |
| `/website_sale/product_configurator/update_combination` | jsonrpc | public | ["POST"] | True | `website_sale` | `WebsiteSaleProductConfiguratorController.website_sale_product_configurator_update_combination` |  |
| `/website_sale/product_configurator/get_optional_products` | jsonrpc | public |  | True | `website_sale` | `WebsiteSaleProductConfiguratorController.website_sale_product_configurator_get_optional_products` |  |
| `/gmc.xml` | http | public |  | True | `website_sale` | `ProductFeed.gmc_feed` | Serve a dynamic XML feed to synchronize the eCommerce products with Google Merchant Center (GMC).  This method generates an XML feed containing information about eCommerce products. The feed is configured via the `product.feed` model, allowing customization such as: - Localization by specifying a la |
| `/my/orders/reorder` | jsonrpc | public |  | True | `website_sale` | `CustomerPortal.my_orders_reorder` | Retrieve reorder content and automatically add products to the cart.  param int order_id: The ID of the sale order to reorder. param str access_token: The access token for the sale order. return: Details of the added products. rtype: dict |
| `/website_sale/get_combination_info` | jsonrpc | public | ["POST"] | True | `website_sale` | `WebsiteSaleVariantController.get_combination_info_website` |  |
| `/sale/create_product_variant` | jsonrpc | public | ["POST"] |  | `website_sale` | `WebsiteSaleVariantController.create_product_variant` | Old product configurator logic, only used by frontend configurator, will be deprecated soon |
| `/website/form/shop.sale.order` | http | public | ["POST"] | True | `website_sale` | `WebsiteSaleForm.website_form_saleorder` |  |
| `/shop/set_click_and_collect_location` | jsonrpc | public |  | True | `website_sale_collect` | `InStoreDelivery.shop_set_click_and_collect_location` | Set the pickup location and the in-store delivery method on the current order or created one.  This route is called from location selector on /product and is distinct from /website_sale/set_pickup_location as the latter is only called from the checkout page after the delivery method is selected.  :p |
| `/shop/compare` | http | public |  | True | `website_sale_comparison` | `WebsiteSaleProductComparison.product_compare` |  |
| `/shop/compare/get_product_data` | jsonrpc | public |  | True | `website_sale_comparison` | `WebsiteSaleProductComparison.get_product_data` |  |
| `/wallet/top_up` | http | user |  | True | `website_sale_loyalty` | `Cart.wallet_top_up` |  |
| `/coupon/<string:code>` | http | public |  | True | `website_sale_loyalty` | `WebsiteSale.activate_coupon` |  |
| `/shop/claimreward` | http | public |  | True | `website_sale_loyalty` | `WebsiteSale.claim_reward` |  |
| `/website_sale_mondialrelay/update_shipping` | jsonrpc | public |  | True | `website_sale_mondialrelay` | `MondialRelay.mondial_relay_update_shipping` |  |
| `/website_sale_mrp/get_unavailable_qty_from_kits` | jsonrpc | public |  | True | `website_sale_mrp` | `WebsiteSaleMrpVariantController.get_unavailable_qty_from_kits` |  |
| `/slides/get_course_products` | jsonrpc | user |  |  | `website_sale_slides` | `WebsiteSaleSlides.get_course_products` | Return a list of the course products values with formatted price. |
| `/shop/add/stock_notification` | jsonrpc | public |  | True | `website_sale_stock` | `WebsiteSaleStock.add_stock_email_notification` |  |
| `/shop/wishlist/add` | jsonrpc | public |  | True | `website_sale_wishlist` | `WebsiteSaleWishlist.add_to_wishlist` |  |
| `/shop/wishlist` | http | public |  | True | `website_sale_wishlist` | `WebsiteSaleWishlist.get_wishlist` |  |
| `/shop/wishlist/remove/<int:wish_id>` | jsonrpc | public |  | True | `website_sale_wishlist` | `WebsiteSaleWishlist.remove_from_wishlist` |  |
| `/shop/wishlist/get_product_ids` | jsonrpc | public |  | True | `website_sale_wishlist` | `WebsiteSaleWishlist.shop_wishlist_get_product_ids` |  |
| `/slides/all` | http | public |  | True | `website_slides` | `WebsiteSlidesLegacy.slides_channel_all` | "All" in < 19 was different from "Home". Both have been merged, but we keep some backward compatibility for saved links, even if the display is going to change a bit. |
| `/slides/all/tag/<string:slug_tags>` | http | public |  | True | `website_slides` | `WebsiteSlidesLegacy.slides_channel_all` | "All" in < 19 was different from "Home". Both have been merged, but we keep some backward compatibility for saved links, even if the display is going to change a bit. |
| `/slides` | http | public |  | True | `website_slides` | `WebsiteSlides.slides_channel` |  |
| `/slides/page/<int:page>` | http | public |  | True | `website_slides` | `WebsiteSlides.slides_channel` |  |
| `/slides/tag/<string:slug_tags>` | http | public |  | True | `website_slides` | `WebsiteSlides.slides_channel` |  |
| `/slides/tag/<string:slug_tags>/page/<int:page>` | http | public |  | True | `website_slides` | `WebsiteSlides.slides_channel` |  |
| `/slides/<int:channel_id>` | http | public |  | True | `website_slides` | `WebsiteSlides.channel` | Will return the rendered page of a course, with optional parameters allowing customization:  :param channel: slide.channel to be rendered. :param channel_id: id of the rendered channel. (*) :param category: slide.slide (should be a category). Filter contents to those     below this category (= secti |
| `/slides/<int:channel_id>/category/<int:category_id>` | http | public |  | True | `website_slides` | `WebsiteSlides.channel` | Will return the rendered page of a course, with optional parameters allowing customization:  :param channel: slide.channel to be rendered. :param channel_id: id of the rendered channel. (*) :param category: slide.slide (should be a category). Filter contents to those     below this category (= secti |
| `/slides/<int:channel_id>/category/<int:category_id>/page/<int:page>` | http | public |  | True | `website_slides` | `WebsiteSlides.channel` | Will return the rendered page of a course, with optional parameters allowing customization:  :param channel: slide.channel to be rendered. :param channel_id: id of the rendered channel. (*) :param category: slide.slide (should be a category). Filter contents to those     below this category (= secti |
| `/slides/<model("slide.channel"):channel>` | http | public |  | True | `website_slides` | `WebsiteSlides.channel` | Will return the rendered page of a course, with optional parameters allowing customization:  :param channel: slide.channel to be rendered. :param channel_id: id of the rendered channel. (*) :param category: slide.slide (should be a category). Filter contents to those     below this category (= secti |
| `/slides/<model("slide.channel"):channel>/page/<int:page>` | http | public |  | True | `website_slides` | `WebsiteSlides.channel` | Will return the rendered page of a course, with optional parameters allowing customization:  :param channel: slide.channel to be rendered. :param channel_id: id of the rendered channel. (*) :param category: slide.slide (should be a category). Filter contents to those     below this category (= secti |
| `/slides/<model("slide.channel"):channel>/tag/<model("slide.tag"):tag>` | http | public |  | True | `website_slides` | `WebsiteSlides.channel` | Will return the rendered page of a course, with optional parameters allowing customization:  :param channel: slide.channel to be rendered. :param channel_id: id of the rendered channel. (*) :param category: slide.slide (should be a category). Filter contents to those     below this category (= secti |
| `/slides/<model("slide.channel"):channel>/tag/<model("slide.tag"):tag>/page/<int:page>` | http | public |  | True | `website_slides` | `WebsiteSlides.channel` | Will return the rendered page of a course, with optional parameters allowing customization:  :param channel: slide.channel to be rendered. :param channel_id: id of the rendered channel. (*) :param category: slide.slide (should be a category). Filter contents to those     below this category (= secti |
| `/slides/<model("slide.channel"):channel>/category/<model("slide.slide"):category>` | http | public |  | True | `website_slides` | `WebsiteSlides.channel` | Will return the rendered page of a course, with optional parameters allowing customization:  :param channel: slide.channel to be rendered. :param channel_id: id of the rendered channel. (*) :param category: slide.slide (should be a category). Filter contents to those     below this category (= secti |
| `/slides/<model("slide.channel"):channel>/category/<model("slide.slide"):category>/page/<int:page>` | http | public |  | True | `website_slides` | `WebsiteSlides.channel` | Will return the rendered page of a course, with optional parameters allowing customization:  :param channel: slide.channel to be rendered. :param channel_id: id of the rendered channel. (*) :param category: slide.slide (should be a category). Filter contents to those     below this category (= secti |
| `/slides/<int:channel_id>/invite` | http | public |  | True | `website_slides` | `WebsiteSlides.slide_channel_invite` | This route is included in the invitation link in email to join / check out the course. It is the main entry point on the attendee's side when sharing or inviting them. As rule of thumb, this will redirect to the course if the rights are given, and to the main /slides page with appropriate error mess |
| `/slides/<int:channel_id>/identify` | http | public |  | True | `website_slides` | `WebsiteSlides.slide_channel_identify_from_invite` | This route redirects invited partners when they click on the login / signup button, when they are asked to login / signup as invited to a course as public user on the course page preview. |
| `/slides/channel/join` | jsonrpc | public |  | True | `website_slides` | `WebsiteSlides.slide_channel_join` |  |
| `/slides/channel/leave` | jsonrpc | user |  | True | `website_slides` | `WebsiteSlides.slide_channel_leave` |  |
| `/slides/channel/tag/search_read` | jsonrpc | user | ["POST"] | True | `website_slides` | `WebsiteSlides.slide_channel_tag_search_read` |  |
| `/slides/channel/tag/group/search_read` | jsonrpc | user | ["POST"] | True | `website_slides` | `WebsiteSlides.slide_channel_tag_group_search_read` |  |
| `/slides/channel/tag/add` | jsonrpc | user | ["POST"] | True | `website_slides` | `WebsiteSlides.slide_channel_tag_add` | Adds a slide channel tag to the specified slide channel.  :param integer channel_id: Channel ID :param list tag_id: Channel Tag ID as first value of list. If id=0, then this is a new tag to                     generate and expects a second list value of the name of the new tag. :param list group_id: |
| `/slides/channel/send_share_email` | jsonrpc | user |  | True | `website_slides` | `WebsiteSlides.slide_channel_send_share_email` |  |
| `/slides/channel/subscribe` | jsonrpc | user |  | True | `website_slides` | `WebsiteSlides.slide_channel_subscribe` |  |
| `/slides/channel/unsubscribe` | jsonrpc | user |  | True | `website_slides` | `WebsiteSlides.slide_channel_unsubscribe` |  |
| `/slides/slide/<model("slide.slide"):slide>` | http | public |  | True | `website_slides` | `WebsiteSlides.slide_view` |  |
| `/slides/slide/<int:slide_id>/share` | http | public |  | True | `website_slides` | `WebsiteSlides.slide_shared_view` |  |
| `/slides/slide/<model("slide.slide"):slide>/pdf_content` | http | public |  | True | `website_slides` | `WebsiteSlides.slide_get_pdf_content` |  |
| `/slides/slide/<int:slide_id>/get_image` | http | public |  | True | `website_slides` | `WebsiteSlides.slide_get_image` |  |
| `/slides/slide/get_html_content` | jsonrpc | public |  | True | `website_slides` | `WebsiteSlides.get_html_content` |  |
| `/slides/slide/<model("slide.slide"):slide>/set_completed` | http | user |  | True | `website_slides` | `WebsiteSlides.slide_set_completed_and_redirect` |  |
| `/slides/slide/set_completed` | jsonrpc | public |  | True | `website_slides` | `WebsiteSlides.slide_set_completed` |  |
| `/slides/slide/<model("slide.slide"):slide>/set_uncompleted` | http | user |  | True | `website_slides` | `WebsiteSlides.slide_set_uncompleted_and_redirect` |  |
| `/slides/slide/set_uncompleted` | jsonrpc | public |  | True | `website_slides` | `WebsiteSlides.slide_set_uncompleted` |  |
| `/slides/slide/like` | jsonrpc | public |  | True | `website_slides` | `WebsiteSlides.slide_like` |  |
| `/slides/slide/archive` | jsonrpc | user |  | True | `website_slides` | `WebsiteSlides.slide_archive` | This route allows channel publishers to archive slides. It has to be done in sudo mode since only restricted_editors can write on slides in ACLs |
| `/slides/slide/toggle_is_preview` | jsonrpc | user |  | True | `website_slides` | `WebsiteSlides.slide_preview` |  |
| `/slides/slide/send_share_email` | jsonrpc | user |  | True | `website_slides` | `WebsiteSlides.slide_send_share_email` |  |
| `/slide_channel_tag/add` | jsonrpc | user | ["POST"] | True | `website_slides` | `WebsiteSlides.slide_channel_tag_create_or_get` |  |
| `/slides/slide/quiz/question_add_or_update` | jsonrpc | user | ["POST"] | True | `website_slides` | `WebsiteSlides.slide_quiz_question_add_or_update` | Add a new question to an existing slide. Completed field of slide.partner link is set to False to make sure that the creator can take the quiz again.  An optional question_id to udpate can be given. In this case question is deleted first before creating a new one to simplify management.  :param inte |
| `/slides/slide/quiz/get` | jsonrpc | public |  | True | `website_slides` | `WebsiteSlides.slide_quiz_get` |  |
| `/slides/slide/quiz/reset` | jsonrpc | user |  | True | `website_slides` | `WebsiteSlides.slide_quiz_reset` |  |
| `/slides/slide/quiz/submit` | jsonrpc | public |  | True | `website_slides` | `WebsiteSlides.slide_quiz_submit` |  |
| `/slides/slide/quiz/save_to_session` | jsonrpc | public |  | True | `website_slides` | `WebsiteSlides.slide_quiz_save_to_session` |  |
| `/slides/category/search_read` | jsonrpc | user | ["POST"] | True | `website_slides` | `WebsiteSlides.slide_category_search_read` |  |
| `/slides/category/add` | http | user | ["POST"] | True | `website_slides` | `WebsiteSlides.slide_category_add` | Adds a category to the specified channel. Slide is added at the end of slide list based on sequence. |
| `/slides/prepare_preview` | jsonrpc | user | ["POST"] | True | `website_slides` | `WebsiteSlides.prepare_preview` | Will attempt to fetch external metadata for this slide from the correct source (YouTube, Google Drive, ...).  To take advantage of the slide business method, we create a temporary slide record before fetching the metadata. This allows a lot of code simplification, since we use "new", it will not cre |
| `/slides/add_slide` | jsonrpc | user | ["POST"] | True | `website_slides` | `WebsiteSlides.create_slide` |  |
| `/slides/tag/search_read` | jsonrpc | user | ["POST"] | True | `website_slides` | `WebsiteSlides.slide_tag_search_read` |  |
| `/slides/embed/<int:slide_id>` | http | public |  | True | `website_slides` | `WebsiteSlides.slides_embed` |  |
| `/slides/embed_external/<int:slide_id>` | http | public |  | True | `website_slides` | `WebsiteSlides.slides_embed_external` |  |
| `/slides_survey/slide/get_certification_url` | http | user |  | True | `website_slides_survey` | `WebsiteSlidesSurvey.slide_get_certification_url` |  |
| `/slides_survey/certification/search_read` | jsonrpc | user | ["POST"] | True | `website_slides_survey` | `WebsiteSlidesSurvey.slides_certification_search_read` |  |
