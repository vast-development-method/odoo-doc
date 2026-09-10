# Mass Mailing (`mailing.mailing`)

**Transport name:** `mailing.mailing`  
**Storage name:** `mailing_mailing`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mass_mailing`  
**Extended by packages:** `marketing_card`, `mass_mailing_crm`, `mass_mailing_sms`, `mass_mailing_sale`

Description: Mass Mailing

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `mail.render.mixin`, `utm.source.mixin`
- Default ordering: `calendar_date DESC`
- Display name field: `subject`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (84)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread |
| `subject` | Subject | single line text |  | required |
| `preview` | Preview | single line text |  | Help: Catchy preview sentence that encourages recipients to open this email. In most inboxes, this is displayed next to the subject. Keep it empty if you prefer the first characters of your email content to appear instead. |
| `email_from` | Send From | single line text |  | computed by rule `_compute_email_from` and stored; precomputed before insertion |
| `favorite` | Favorite | boolean |  | changes are tracked in the message thread; not copied on duplication |
| `favorite_date` | Favorite Date | date and time |  | computed by rule `_compute_favorite_date` and stored; not copied on duplication; Help: When this mailing was added in the favorites |
| `sent_date` | Sent Date | date and time |  | not copied on duplication |
| `schedule_type` | Schedule | selection |  | required; default `now` |
| `schedule_date` | Scheduled for | date and time |  | computed by rule `_compute_schedule_date` and stored; changes are tracked in the message thread |
| `calendar_date` | Calendar Date | date and time |  | computed by rule `_compute_calendar_date` and stored; not copied on duplication; Help: Date at which the mailing was or will be sent. |
| `body_arch` | Body | rich text |  |  |
| `body_html` | Body converted to be sent by mail | rich text |  |  |
| `is_body_empty` | Is Body Empty | boolean |  | computed by rule `_compute_is_body_empty` (not stored) |
| `attachment_ids` | Attachments | many to many | `ir.attachment` | association table `mass_mailing_ir_attachments_rel` |
| `keep_archives` | Keep Archives | boolean |  |  |
| `campaign_id` | campaign tracking parameter Campaign | many to one | `utm.campaign` | indexed; on delete of the target: set null |
| `medium_id` | Medium | many to one | `utm.medium` | computed by rule `_compute_medium_id` and stored; on delete of the target: restrict; Help: UTM Medium: delivery method (email, sms, ...) |
| `state` | Status | selection |  | required; default `draft`; changes are tracked in the message thread; not copied on duplication |
| `color` | Color Index | integer |  |  |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread |
| `mailing_type` | Mailing Type | selection |  | required; default `mail`; on delete of the target: {"sms": "set default"}; extended by packages `mass_mailing_sms` |
| `mailing_type_description` | Mailing Type Description | single line text |  | computed by rule `_compute_mailing_type_description` (not stored) |
| `reply_to_mode` | Reply-To Mode | selection |  | computed by rule `_compute_reply_to_mode` and stored; Help: Thread: replies go to target document. Email: replies are routed to a given email. |
| `reply_to` | Reply To | single line text |  | computed by rule `_compute_reply_to` and stored; Help: Preferred Reply-To Address |
| `mailing_model_real` | Recipients Real Model | single line text |  | computed by rule `_compute_mailing_model_real` (not stored) |
| `mailing_model_id` | Recipients Model | many to one | `ir.model` | required; computed by rule `_compute_mailing_model_id` and stored; default computed dynamically (lambda self: self.env.ref('mass_mailing.model_mailing_list').id); on delete of the target: cascade; restricted by domain `[["is_mailing_enabled", "=", true]]`; extended by packages `marketing_card` |
| `mailing_model_name` | Recipients Model Name | single line text |  | read only; related through path `mailing_model_id.model` |
| `mailing_on_mailing_list` | Based on Mailing Lists | boolean |  | computed by rule `_compute_mailing_on_mailing_list` (not stored) |
| `mailing_domain` | Domain | single line text |  | computed by rule `_compute_mailing_domain` and stored |
| `mail_server_available` | Mail Server Available | boolean |  | computed by rule `_compute_mail_server_available` (not stored); Help: Technical field used to know if the user has activated the outgoing mail server option in the settings |
| `mail_server_id` | Mail Server | many to one | `ir.mail_server` | default computed dynamically (_get_default_mail_server_id); indexed (btree_not_null); Help: Use a specific mail server in priority. Otherwise Odoo relies on the first outgoing mail server available (based on their sequencing) as it does for normal mails. |
| `contact_list_ids` | Mailing Lists | many to many | `mailing.list` | association table `mail_mass_mailing_list_rel` |
| `use_exclusion_list` | Use Exclusion List | boolean |  | default `True`; not copied on duplication; Help: Prevent sending messages to blacklisted contacts. Disable only when absolutely necessary. |
| `mailing_filter_id` | Favorite Filter | many to one | `mailing.filter` | computed by rule `_compute_mailing_filter_id` and stored; restricted by domain `[('mailing_model_name', '=', mailing_model_name)]` |
| `mailing_filter_domain` | Favorite filter domain | single line text |  | related through path `mailing_filter_id.mailing_domain` |
| `mailing_filter_count` | # Favorite Filters | integer |  | computed by rule `_compute_mailing_filter_count` (not stored) |
| `ab_testing_completed` | Ab Testing Completed | boolean |  | related through path `campaign_id.ab_testing_completed` |
| `ab_testing_description` | A/B Testing Description | rich text |  | computed by rule `_compute_ab_testing_description` (not stored) |
| `ab_testing_enabled` | Allow A/B Testing | boolean |  | default ; Help: If checked, recipients will be mailed only once for the whole campaign. This lets you send different mailings to randomly selected recipients and test the effectiveness of the mailings, without causing duplicate messages. |
| `ab_testing_is_winner_mailing` | Is the Winner of its Campaign | boolean |  | computed by rule `_compute_ab_testing_is_winner_mailing` (not stored) |
| `ab_testing_mailings_count` | Ab Testing Mailings Count | integer |  | related through path `campaign_id.ab_testing_mailings_count` |
| `ab_testing_pc` | A/B Testing percentage | integer |  | default `10`; Help: Percentage of the contacts that will be mailed. Recipients will be chosen randomly. |
| `ab_testing_schedule_datetime` | Ab Testing Schedule Datetime | date and time |  | related through path `campaign_id.ab_testing_schedule_datetime`; default computed dynamically (lambda self: fields.Datetime.now() + relativedelta(days=1)) |
| `ab_testing_winner_selection` | Ab Testing Winner Selection | selection |  | related through path `campaign_id.ab_testing_winner_selection`; default `opened_ratio` |
| `is_ab_test_sent` | Is Ab Test Sent | boolean |  | computed by rule `_compute_is_ab_test_sent` (not stored) |
| `kpi_mail_required` | key performance indicator mail required | boolean |  | not copied on duplication |
| `mailing_trace_ids` | Emails Statistics | one to many | `mailing.trace` | inverse field `mass_mailing_id` |
| `total` | Total | integer |  | computed by rule `_compute_total` (not stored) |
| `scheduled` | Scheduled | integer |  | computed by rule `_compute_statistics` (not stored) |
| `expected` | Expected | integer |  | computed by rule `_compute_statistics` (not stored) |
| `canceled` | Canceled | integer |  | computed by rule `_compute_statistics` (not stored) |
| `sent` | Sent | integer |  | computed by rule `_compute_statistics` (not stored) |
| `process` | Process | integer |  | computed by rule `_compute_statistics` (not stored) |
| `pending` | Pending | integer |  | computed by rule `_compute_statistics` (not stored) |
| `delivered` | Delivered | integer |  | computed by rule `_compute_statistics` (not stored) |
| `opened` | Opened | integer |  | computed by rule `_compute_statistics` (not stored) |
| `clicked` | Clicked | integer |  | computed by rule `_compute_statistics` (not stored) |
| `replied` | Replied | integer |  | computed by rule `_compute_statistics` (not stored) |
| `bounced` | Bounced | integer |  | computed by rule `_compute_statistics` (not stored) |
| `failed` | Failed | integer |  | computed by rule `_compute_statistics` (not stored) |
| `received_ratio` | Received Ratio | float |  | computed by rule `_compute_statistics` (not stored) |
| `opened_ratio` | Opened Ratio | float |  | computed by rule `_compute_statistics` (not stored) |
| `replied_ratio` | Replied Ratio | float |  | computed by rule `_compute_statistics` (not stored) |
| `bounced_ratio` | Bounced Ratio | float |  | computed by rule `_compute_statistics` (not stored) |
| `clicks_ratio` | Number of Clicks | float |  | computed by rule `_compute_clicks_ratio` (not stored) |
| `link_trackers_count` | Link Trackers Count | integer |  | computed by rule `_compute_link_trackers_count` (not stored) |
| `next_departure` | Scheduled date | date and time |  | computed by rule `_compute_next_departure` (not stored) |
| `next_departure_is_past` | Next Departure Is Past | boolean |  | computed by rule `_compute_next_departure` (not stored) |
| `warning_message` | Warning Message | single line text |  | computed by rule `_compute_warning_message` (not stored); Help: Warning message displayed in the mailing form view |
| `card_requires_sync_count` | Card Requires Sync Count | integer |  | computed by rule `_compute_card_requires_sync_count` (not stored) |
| `card_campaign_id` | Card Campaign | many to one | `card.campaign` | indexed (btree_not_null) |
| `use_leads` | Use Leads | boolean |  | computed by rule `_compute_use_leads` (not stored) |
| `crm_lead_count` | Leads/Opportunities Count | integer |  | computed by rule `_compute_crm_lead_count` (not stored) |
| `sms_subject` | Title | single line text |  | related through path `subject`; Help: For an email, the subject your recipients will see in their inbox. For an SMS, the internal title of the message. |
| `body_plaintext` | text message Body | multi line text |  | computed by rule `_compute_body_plaintext` and stored |
| `sms_template_id` | text message Template | many to one | `sms.template` | on delete of the target: set null |
| `sms_has_insufficient_credit` | Insufficient in-app purchase credits | boolean |  | computed by rule `_compute_sms_has_iap_failure` (not stored) |
| `sms_has_unregistered_account` | Unregistered in-app purchase account | boolean |  | computed by rule `_compute_sms_has_iap_failure` (not stored) |
| `sms_force_send` | Send Directly | boolean |  | Help: Immediately send the SMS Mailing instead of queuing up. Use at your own risk. |
| `sms_allow_unsubscribe` | Include opt-out link | boolean |  | default  |
| `ab_testing_sms_winner_selection` | Ab Testing Text message Winner Selection | selection |  | related through path `campaign_id.ab_testing_sms_winner_selection`; default `clicks_ratio` |
| `ab_testing_mailings_sms_count` | Ab Testing Mailings Text message Count | integer |  | related through path `campaign_id.ab_testing_mailings_sms_count` |
| `sale_quotation_count` | Quotation Count | integer |  | computed by rule `_compute_sale_quotation_count` (not stored) |
| `sale_invoiced_amount` | Invoiced Amount | integer |  | computed by rule `_compute_sale_invoiced_amount` (not stored) |

## Selection values

### `schedule_type` (Schedule)

| Value | Label |
|---|---|
| `now` | Send now |
| `scheduled` | Send on |

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `in_queue` | In Queue |
| `sending` | Sending |
| `done` | Sent |

### `mailing_type` (Mailing Type)

| Value | Label |
|---|---|
| `mail` | Email |
| `sms` | SMS |

### `reply_to_mode` (Reply-To Mode)

| Value | Label |
|---|---|
| `update` | Recipient Followers |
| `new` | Specified Email Address |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_percentage_valid` | Constraint | `CHECK(ab_testing_pc >= 0 AND ab_testing_pc <= 100)` | The A/B Testing Percentage needs to be between 0 and 100% | `mass_mailing` |
| `_email_from` | Constraint | `CHECK(email_from IS NOT NULL OR mailing_type != 'mail')` | email from is required for mailing | `mass_mailing` |

## Operations (112)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mass_mailing_sms`, `mass_mailing` | model |  |
| `_get_default_mail_server_id` | preparation rule | self | `mass_mailing` | model |  |
| `_check_mailing_filter_model` | validation | self | `mass_mailing` | constrains: `mailing_model_id`, `mailing_filter_id` | Check that if the favorite filter is set, it must contain the same recipient model as mailing |
| `_compute_ab_testing_is_winner_mailing` | computation | self | `mass_mailing` | depends: `campaign_id.ab_testing_winner_mailing_id` |  |
| `_compute_email_from` | computation | self | `mass_mailing` | depends: `mail_server_id`, `create_uid` |  |
| `_compute_favorite_date` | computation | self | `mass_mailing` | depends: `favorite` |  |
| `_compute_total` | computation | self | `mass_mailing` |  |  |
| `_compute_clicks_ratio` | computation | self | `mass_mailing` |  |  |
| `_compute_statistics` | computation | self | `mass_mailing` |  | Compute statistics of the mass mailing |
| `_compute_next_departure` | computation | self | `mass_mailing` | depends: `schedule_date`, `state` |  |
| `_compute_link_trackers_count` | computation | self | `mass_mailing` |  |  |
| `_compute_warning_message` | computation | self | `mass_mailing` | depends: `email_from`, `mail_server_id` |  |
| `_compute_medium_id` | computation | self | `mass_mailing_sms`, `mass_mailing` | depends: `mailing_type` |  |
| `_compute_reply_to_mode` | computation | self | `mass_mailing` | depends: `mailing_model_id` | For main models not really using chatter to gather answers (contacts and mailing contacts), set reply-to as email-based. Otherwise answers by default go on the original discussion thread (business document). Note that mailing_model being mailing.list means contacting mailing.contact (see mailing_model_name versus mailing_model_real). |
| `_compute_reply_to` | computation | self | `mass_mailing` | depends: `reply_to_mode` |  |
| `_compute_mailing_filter_count` | computation | self | `mass_mailing` | depends: `mailing_model_id`, `mailing_domain` |  |
| `_compute_mailing_model_real` | computation | self | `mass_mailing` | depends: `mailing_model_id` |  |
| `_compute_mailing_on_mailing_list` | computation | self | `mass_mailing` | depends: `mailing_model_id` |  |
| `_compute_mailing_domain` | computation | self | `mass_mailing` | depends: `mailing_model_id`, `contact_list_ids`, `mailing_type`, `mailing_filter_id` |  |
| `_compute_mailing_filter_id` | computation | self | `mass_mailing` | depends: `mailing_model_name` |  |
| `_compute_schedule_date` | computation | self | `mass_mailing` | depends: `schedule_type` |  |
| `_compute_calendar_date` | computation | self | `mass_mailing` | depends: `state`, `schedule_date`, `sent_date`, `next_departure` |  |
| `_compute_is_body_empty` | computation | self | `mass_mailing` | depends: `body_arch` |  |
| `_compute_mail_server_available` | computation | self | `mass_mailing` |  |  |
| `_compute_render_model` | computation | self | `mass_mailing` | depends: `mailing_model_real` |  |
| `_compute_mailing_type_description` | computation | self | `mass_mailing` | depends: `mailing_type` |  |
| `_compute_ab_testing_description` | computation | self | `mass_mailing` | depends: |  |
| `_compute_is_ab_test_sent` | computation | self | `mass_mailing` | depends: `campaign_id.mailing_mail_ids.state` |  |
| `_get_ab_testing_description_modifying_fields` | preparation rule | self | `mass_mailing_sms`, `mass_mailing` |  |  |
| `create` | lifecycle override | self, vals_list | `mass_mailing_sms`, `mass_mailing` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mass_mailing` |  |  |
| `_create_ab_testing_utm_campaigns` | internal rule | self | `mass_mailing` |  | Creates the A/B test campaigns for the mailings that do not have campaign set already |
| `_fix_attachment_ownership` | internal rule | self | `mass_mailing` |  |  |
| `copy_data` | lifecycle override | self, default | `mass_mailing` |  |  |
| `action_set_favorite` | user action | self | `mass_mailing` |  | Add the current mailing in the favorites list. |
| `action_remove_favorite` | user action | self | `mass_mailing` |  | Remove the current mailing from the favorites list. |
| `action_duplicate` | user action | self | `mass_mailing` |  |  |
| `action_test` | user action | self | `mass_mailing_sms`, `mass_mailing` |  |  |
| `action_launch` | user action | self | `mass_mailing` |  |  |
| `action_reload` | user action | self | `mass_mailing` |  |  |
| `action_schedule` | user action | self | `mass_mailing` |  |  |
| `action_put_in_queue` | user action | self | `marketing_card`, `mass_mailing` |  | Detect mismatches before scheduling. |
| `action_cancel` | user action | self | `mass_mailing` |  |  |
| `action_retry_failed` | user action | self | `mass_mailing_sms`, `mass_mailing` |  | Remove all failed emails and their traces, and try sending them again. |
| `action_view_link_trackers` | user action | self | `mass_mailing` |  |  |
| `action_view_traces_scheduled` | user action | self | `mass_mailing` |  |  |
| `action_view_traces_canceled` | user action | self | `mass_mailing` |  |  |
| `action_view_traces_failed` | user action | self | `mass_mailing` |  |  |
| `action_view_traces_process` | user action | self | `mass_mailing` |  |  |
| `action_view_traces_sent` | user action | self | `mass_mailing` |  |  |
| `_action_view_traces_filtered` | internal rule | self, view_filter | `mass_mailing_sms`, `mass_mailing` |  |  |
| `action_view_clicked` | user action | self | `mass_mailing` |  |  |
| `action_view_opened` | user action | self | `mass_mailing` |  |  |
| `action_view_replied` | user action | self | `mass_mailing` |  |  |
| `action_view_bounced` | user action | self | `mass_mailing` |  |  |
| `action_view_delivered` | user action | self | `mass_mailing` |  |  |
| `_action_view_documents_filtered` | internal rule | self, view_filter | `mass_mailing` |  |  |
| `action_view_mailing_contacts` | user action | self | `mass_mailing` |  | Show the mailing contacts who are in a mailing list selected for this mailing. |
| `action_fetch_favorites` | user action | self, extra_domain | `mass_mailing` | model | Return all mailings set as favorite and skip mailings with empty body.  Return archived mailing templates as well, so the user can archive the templates while keeping using it, without cluttering the Kanban view if they're a lot of templates. |
| `action_compare_versions` | user action | self | `mass_mailing` |  |  |
| `action_send_winner_mailing` | user action | self | `mass_mailing` |  | Send the winner mailing based on the winner selection field. This action is used in 2 cases:  - When the user clicks on a button to send the winner mailing. There is only one mailing in self - When the cron is executed to send winner mailing based on the A/B testing schedule datetime. In this   case 'self' contains all the mailing for the campaigns so we just need to take the first to determine the   winner.  If the winner mailing is computed automatically, we sudo the mailings of the campaign in order to sort correctly the mailings based on the selection that can be used with sub-modules like |
| `action_select_as_winner` | user action | self | `mass_mailing` |  |  |
| `_get_ab_testing_description_values` | preparation rule | self | `mass_mailing_sms`, `mass_mailing` |  |  |
| `_get_ab_testing_siblings_mailings` | preparation rule | self | `mass_mailing_sms`, `mass_mailing` |  |  |
| `_get_ab_testing_winner_selection` | preparation rule | self | `mass_mailing_sms`, `mass_mailing` |  |  |
| `_get_default_ab_testing_campaign_values` | preparation rule | self, values | `mass_mailing_sms`, `mass_mailing` |  |  |
| `_get_opt_out_list` | preparation rule | self | `mass_mailing` |  | Give list of opt-outed emails, depending on specific model-based computation if available.  :returns: opt-outed emails, preferably normalized (aka not records) |
| `_get_link_tracker_values` | preparation rule | self | `mass_mailing` |  |  |
| `_get_seen_list` | preparation rule | self | `mass_mailing` |  | Returns a set of emails already targeted by current mailing/campaign (no duplicates) |
| `_get_seen_list_extra` | preparation rule | self | `mass_mailing` |  |  |
| `_get_mass_mailing_context` | preparation rule | self | `mass_mailing` |  | Returns extra context items with pre-filled blacklist and seen list for massmailing |
| `_get_recipients` | preparation rule | self | `mass_mailing` |  |  |
| `_get_recipients_domain` | preparation rule | self | `marketing_card`, `mass_mailing` |  | Overridable getter used to get the domain of the recipients at the time of sending. |
| `_get_remaining_recipients` | preparation rule | self | `mass_mailing` |  |  |
| `_get_recipient_base_url` | preparation rule | self | `mass_mailing` |  | Base URL of the recipient record passed in context when sending, so links stay on the recipient's website in multi-website setups (see mail.mail). Only a recordset is used, never a plain value, so a context set from an RPC call cannot redirect the links to another host. |
| `_get_unsubscribe_oneclick_url` | preparation rule | self, email_to, res_id | `mass_mailing` |  |  |
| `_get_unsubscribe_url` | preparation rule | self, email_to, res_id | `mass_mailing` |  |  |
| `_get_view_url` | preparation rule | self, email_to, res_id | `mass_mailing` |  |  |
| `action_send_mail` | user action | self, res_ids | `marketing_card`, `mass_mailing` |  |  |
| `_action_send_mail` | internal rule | self, res_ids | `mass_mailing_sms`, `mass_mailing` |  |  |
| `convert_links` | operation | self | `mass_mailing_sms`, `mass_mailing` |  |  |
| `_process_mass_mailing_queue` | background operation | self | `mass_mailing` | model |  |
| `_action_send_statistics` | internal rule | self | `mass_mailing` |  | Send an email to the responsible of each finished mailing with the statistics. |
| `_prepare_statistics_email_values` | preparation rule | self | `mass_mailing_crm`, `mass_mailing_sale`, `mass_mailing_sms`, `mass_mailing` |  | Return some statistics that will be displayed in the mailing statistics email.  Each item in the returned list will be displayed as a table, with a title and 1, 2 or 3 columns. |
| `_get_pretty_mailing_type` | preparation rule | self | `mass_mailing_sms`, `mass_mailing` |  |  |
| `_generate_mailing_report_token` | internal rule | self, user_id | `mass_mailing` |  | Generate a secure token for this user. It allows to opt out from mailing reports while keeping some security in that process. |
| `_convert_inline_images_to_urls` | internal rule | self, html_content | `mass_mailing` |  | Find inline base64 encoded images, make an attachement out of them and replace the inline image with an url to the attachement. Find VML v:image elements, crop their source images, make an attachement out of them and replace their source with an url to the attachement. |
| `_create_attachments_from_inline_images` | internal rule | self, b64images | `mass_mailing` |  |  |
| `_get_default_mailing_domain` | preparation rule | self | `mass_mailing` |  |  |
| `_get_image_by_url` | preparation rule | self, url, session | `mass_mailing` |  |  |
| `_parse_mailing_domain` | internal rule | self | `mass_mailing` |  |  |
| `_generate_mailing_recipient_token` | internal rule | self, document_id, email | `mass_mailing` |  | Generate a secure token for a given mailing and recipient (based on their email). This allows notably to unsubscribe from the mailing or to blacklist their email entirely without need of a user account.  :param int document_id: ID of the business document on which mailing   is performed; :param str email: recipient email, used to unsubscribe / blacklist; |
| `_check_mailing_domain` | validation | self | `marketing_card` | constrains: `card_campaign_id`, `mailing_domain`, `mailing_model_id` |  |
| `_compute_mailing_model_id` | computation | self | `marketing_card` | depends: `card_campaign_id` |  |
| `_compute_card_requires_sync_count` | computation | self | `marketing_card` | depends: `card_campaign_id` | Check if there's any missing or outdated card. |
| `action_update_cards` | user action | self | `marketing_card` |  | Update the cards in batches, commiting after each batch. |
| `_compute_use_leads` | computation | self | `mass_mailing_crm` |  |  |
| `_compute_crm_lead_count` | computation | self | `mass_mailing_crm` |  |  |
| `action_redirect_to_leads_and_opportunities` | user action | self | `mass_mailing_crm` |  |  |
| `_compute_body_plaintext` | computation | self | `mass_mailing_sms` | depends: `sms_template_id`, `mailing_type` |  |
| `_compute_sms_has_iap_failure` | computation | self | `mass_mailing_sms` | depends: `mailing_trace_ids.failure_type` |  |
| `action_retry_failed_sms` | user action | self | `mass_mailing_sms` |  |  |
| `action_buy_sms_credits` | user action | self | `mass_mailing_sms` |  |  |
| `_get_opt_out_list_sms` | preparation rule | self | `mass_mailing_sms` |  | Give list of opt-outed records, depending on specific model-based computation if available.  :returns: opt-outed record IDs :rtype: list |
| `_get_seen_list_sms` | preparation rule | self | `mass_mailing_sms` |  | Returns a set of emails already targeted by current mailing/campaign (no duplicates) |
| `_send_sms_get_composer_values` | internal rule | self, res_ids | `mass_mailing_sms` |  |  |
| `action_send_sms` | user action | self, res_ids | `mass_mailing_sms` |  |  |
| `get_sms_link_replacements_placeholders` | operation | self | `mass_mailing_sms` |  | Get placeholders for replaced links in sms widget for accurate computation of sms counts.  Reminders and assumptions:  * Links wille be transformed to the format ``"[base_url]/r/[link_tracker_code]/s/[sms_id]"``. * unsubscribe is formatted as: ``"STOP SMS : [base_url]/sms/[mailing_id]/[trace_code]"``.  :returns: Character counts used for links, formatted as ``{link: str, unsubscribe: str}``. |
| `_compute_sale_quotation_count` | computation | self | `mass_mailing_sale` | depends: `mailing_domain` |  |
| `_compute_sale_invoiced_amount` | computation | self | `mass_mailing_sale` | depends: `mailing_domain` |  |
| `action_redirect_to_quotations` | user action | self | `mass_mailing_sale` |  |  |
| `action_redirect_to_invoiced` | user action | self | `mass_mailing_sale` |  |  |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_mailing_filter_model` | ValidationError | The saved filter targets different recipients and is incompatible with this mailing. | `mass_mailing` |
| `write` | ValidationError | A campaign should be set when A/B test is enabled | `mass_mailing` |
| `action_send_winner_mailing` | ValidationError | No mailing for this A/B testing campaign has been sent yet! Send one first and try again later. | `mass_mailing` |
| `_action_send_mail` | UserError | There are no recipients selected. | `mass_mailing` |
| `_check_mailing_domain` | ValidationError | Card Campaign Mailing should target model %(model_name)s | `marketing_card` |
| `action_put_in_queue` | UserError | You should update all the cards for %(mailing)s before scheduling a mailing. | `marketing_card` |
| `action_send_mail` | UserError | You should update all the cards for %(mailing)s before scheduling a mailing. | `marketing_card` |
| `_get_seen_list_sms` | UserError | Unsupported %s for mass SMS | `mass_mailing_sms` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `base.group_system` | yes | yes | yes | yes | `mass_mailing` |

## Views (13)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `marketing_card.mailing_mailing_view_form_inherit_marketing_card` | xpath | `mass_mailing.view_mail_mass_mailing_form` |  |  |  | `marketing_card` |
| `mass_mailing.view_mail_mass_mailing_search` | search |  | `name`, `campaign_id` |  | `My Mailings`, `filter_sent_date`, `A/B Tests`, `A/B Tests to review`, `Archived`, `Status`, `Sent By`, `Mailing List`, `Sent Period` | `mass_mailing` |
| `mass_mailing.view_mail_mass_mailing_tree` | list |  | `calendar_date`, `subject`, `mailing_model_id`, `user_id`, `ab_testing_enabled`, `campaign_id`, `sent`, `received_ratio`, `opened_ratio`, `bounced_ratio`, `clicks_ratio`, `replied_ratio`, `state` |  |  | `mass_mailing` |
| `mass_mailing.view_mail_mass_mailing_form` | form |  | `state`, `canceled`, `scheduled`, `process`, `sent`, `failed`, `next_departure`, `next_departure_is_past`, `next_departure_is_past`, `schedule_type`, `warning_message`, `opened_ratio`, `replied_ratio`, `clicks_ratio`, `received_ratio`, `bounced_ratio`, `link_trackers_count`, `active`, `create_uid`, `mailing_type`, `subject`, `mailing_model_id`, `contact_list_ids`, `mailing_filter_id`, `mailing_filter_count`, `mailing_model_name`, `mailing_on_mailing_list`, `mailing_model_real`, `mailing_domain`, `body_arch`, `body_html`, `is_body_empty`, `ab_testing_enabled`, `ab_testing_pc`, `ab_testing_winner_selection`, `ab_testing_schedule_datetime`, `is_ab_test_sent`, `ab_testing_mailings_count`, `ab_testing_completed`, `ab_testing_description`, `preview`, `email_from`, `reply_to_mode`, `reply_to`, `attachment_ids`, `campaign_id`, `medium_id`, `source_id`, `user_id`, `mail_server_available`, `name`, `mail_server_id`, `keep_archives`, `use_exclusion_list` | `Send`, `Schedule`, `Duplicate`, `Test`, `Cancel`, `Retry`, `Add to Templates`, `Remove from Templates`, `action_view_traces_canceled`, `action_view_traces_scheduled`, `action_view_traces_process`, `action_view_traces_sent`, `action_view_traces_failed`, `action_reload`, `action_reload`, `action_view_opened`, `action_view_replied`, `action_view_clicked`, `action_view_delivered`, `action_view_bounced`, `action_view_link_trackers`, `action_view_mailing_contacts`, `action_compare_versions`, `action_duplicate`, `action_send_winner_mailing`, `action_select_as_winner`, `action_duplicate` |  | `mass_mailing` |
| `mass_mailing.view_mail_mass_mailing_kanban` | kanban |  | `mailing_on_mailing_list`, `next_departure`, `active`, `color`, `subject`, `campaign_id`, `mailing_model_id`, `sent_date`, `schedule_date`, `total`, `mailing_model_id`, `user_id` |  |  | `mass_mailing` |
| `mass_mailing.mailing_mailing_view_calendar` | calendar |  | `mailing_model_id`, `user_id`, `state` |  |  | `mass_mailing` |
| `mass_mailing_crm.mailing_mailing_view_form` | xpath | `mass_mailing.view_mail_mass_mailing_form` | `use_leads`, `crm_lead_count` | `action_redirect_to_leads_and_opportunities` |  | `mass_mailing_crm` |
| `mass_mailing_sale.mailing_mailing_view_form` | xpath | `mass_mailing.view_mail_mass_mailing_form` | `sale_quotation_count`, `sale_invoiced_amount` | `action_redirect_to_quotations`, `action_redirect_to_invoiced` |  | `mass_mailing_sale` |
| `mass_mailing_sms.mailing_mailing_view_search_sms` | xpath | `mass_mailing.view_mail_mass_mailing_search` |  |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_mailing_view_form_sms` | xpath | `mass_mailing.view_mail_mass_mailing_form` |  |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_mailing_view_form_mixed` | xpath | `mass_mailing_sms.mailing_mailing_view_form_sms` |  |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_mailing_view_kanban_sms` | xpath | `mass_mailing.view_mail_mass_mailing_kanban` | `sms_has_insufficient_credit`, `sms_has_unregistered_account` |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_mailing_view_tree_sms` | list |  | `calendar_date`, `subject`, `mailing_type`, `mailing_model_id`, `user_id`, `campaign_id`, `ab_testing_enabled`, `sent`, `clicked`, `bounced`, `state` |  |  | `mass_mailing_sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_mailing_action_mail` | Mailings | list,kanban,form,calendar | `[('mailing_type', '=', 'mail')]` | `{                     'search_default_assigned_to_me': 1,                     'default_user_id': uid,                     'default_mailing_type': 'mail',             }` |  | `mass_mailing` |
| `mass_mailing.action_view_mass_mailings_from_campaign` | Mailings | kanban,list,form,calendar | `[('mailing_type', '=', 'mail')]` | `{                 'search_default_assigned_to_me': 1,                 'search_default_campaign_id': [active_id],                 'default_campaign_id': active_id,                 'default_user_id': uid,             }` |  | `mass_mailing` |
| `mass_mailing.action_create_mass_mailings_from_campaign` | Mailings | form,kanban,list |  | `{                 'search_default_assigned_to_me': 1,                 'search_default_campaign_id': [active_id],                 'default_campaign_id': active_id,                 'default_user_id': uid,             }` |  | `mass_mailing` |
| `mass_mailing.action_ab_testing_open_winner_mailing` | A/B Test Winner | form |  |  |  | `mass_mailing` |
| `mass_mailing_sms.mailing_mailing_action_sms` | SMS Marketing | list,kanban,form,calendar,graph | `[('mailing_type', '=', 'sms')]` | `{                 'search_default_assigned_to_me': 1,                 'default_user_id': uid,                 'default_mailing_type': 'sms',                 'mailing_sms': True         }` |  | `mass_mailing_sms` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mass_mailing.ir_cron_mass_mailing_queue` | Mail Marketing: Process queue | 1 days | `_process_mass_mailing_queue` | 6 |

Machine-readable definition: `../../../schemas/data/entities/mailing.mailing.json`; views: `../../../schemas/interfaces/views/mailing.mailing.json`.
