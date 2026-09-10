# Send text message Wizard (`sms.composer`)

**Transport name:** `sms.composer`  
**Storage name:** `sms_composer`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sms`  
**Extended by packages:** `mass_mailing_sms`, `sms_twilio`

Description: Send SMS Wizard

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `composition_mode` | Composition Mode | selection |  | required; computed by rule `_compute_composition_mode` and stored; precomputed before insertion |
| `res_model` | Document Model Name | single line text |  |  |
| `res_model_description` | Document Model Description | single line text |  | computed by rule `_compute_res_model_description` (not stored) |
| `res_id` | Document identifier | integer |  |  |
| `res_ids` | Document identifiers | single line text |  |  |
| `res_ids_count` | Visible records count | integer |  | computed by rule `_compute_res_ids_count` (not stored); Help: Number of recipients that will receive the SMS if sent in mass mode, without applying the Active Domain value |
| `comment_single_recipient` | Single Mode | boolean |  | computed by rule `_compute_comment_single_recipient` (not stored); Help: Indicates if the SMS composer targets a single specific recipient |
| `mass_keep_log` | Keep a note on document | boolean |  | default `True` |
| `mass_force_send` | Send directly | boolean |  | default  |
| `use_exclusion_list` | Use Exclusion List | boolean |  | default `True`; not copied on duplication; Help: Prevent sending messages to blacklisted contacts. Disable only when absolutely necessary. |
| `recipient_valid_count` | # Valid recipients | integer |  | computed by rule `_compute_recipients` (not stored) |
| `recipient_invalid_count` | # Invalid recipients | integer |  | computed by rule `_compute_recipients` (not stored) |
| `recipient_single_description` | Recipients (Partners) | multi line text |  | computed by rule `_compute_recipient_single_non_stored` (not stored) |
| `recipient_single_number` | Stored Recipient Number | single line text |  | computed by rule `_compute_recipient_single_non_stored` (not stored) |
| `recipient_single_number_itf` | Recipient Number | single line text |  | computed by rule `_compute_recipient_single_stored` and stored; Help: Phone number of the recipient. If changed, it will be recorded on recipient's profile. |
| `recipient_single_valid` | Is valid | boolean |  | computed by rule `_compute_recipient_single_valid` (not stored) |
| `number_field_name` | Number Field | single line text |  |  |
| `numbers` | Recipients (Numbers) | single line text |  |  |
| `sanitized_numbers` | Sanitized Number | single line text |  | computed by rule `_compute_sanitized_numbers` (not stored) |
| `template_id` | Use Template | many to one | `sms.template` | restricted by domain `[('model', '=', res_model)]` |
| `body` | Message | multi line text |  | required; computed by rule `_compute_body` and stored; precomputed before insertion |
| `mass_sms_allow_unsubscribe` | Include opt-out link | boolean |  | default `True` |
| `mailing_id` | Mailing | many to one | `mailing.mailing` |  |
| `utm_campaign_id` | Campaign | many to one | `utm.campaign` | on delete of the target: set null |

## Selection values

### `composition_mode` (Composition Mode)

| Value | Label |
|---|---|
| `numbers` | Send to numbers |
| `comment` | Post on a document |
| `mass` | Send SMS in batch |

## Operations (35)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `sms` | model |  |
| `_compute_composition_mode` | computation | self | `sms` | depends: `res_ids_count`; depends_context: `sms_composition_mode` |  |
| `_compute_res_model_description` | computation | self | `sms` | depends: `res_model` |  |
| `_compute_res_ids_count` | computation | self | `sms` | depends: `res_model`, `res_id`, `res_ids` |  |
| `_compute_comment_single_recipient` | computation | self | `sms` | depends: `res_id`, `composition_mode` |  |
| `_compute_recipients` | computation | self | `sms` | depends: `res_model`, `res_id`, `res_ids`, `composition_mode`, `number_field_name`, `sanitized_numbers` |  |
| `_compute_recipient_single_stored` | computation | self | `sms` | depends: `res_model`, `number_field_name` |  |
| `_compute_recipient_single_non_stored` | computation | self | `sms` | depends: `res_model`, `number_field_name` |  |
| `_compute_recipient_single_valid` | computation | self | `sms` | depends: `recipient_single_number`, `recipient_single_number_itf` |  |
| `_compute_sanitized_numbers` | computation | self | `sms` | depends: `numbers`, `res_model`, `res_id` |  |
| `_compute_body` | computation | self | `sms` | depends: `composition_mode`, `res_model`, `res_id`, `template_id` |  |
| `action_send_sms` | user action | self | `sms` |  |  |
| `action_send_sms_mass_now` | user action | self | `sms` |  |  |
| `_action_send_sms` | internal rule | self | `sms` |  |  |
| `_action_send_sms_numbers` | internal rule | self | `sms` |  |  |
| `_action_send_sms_comment_single` | internal rule | self, records | `sms` |  |  |
| `_action_send_sms_comment` | internal rule | self, records | `sms` |  |  |
| `_action_send_sms_mass` | internal rule | self, records | `sms` |  |  |
| `_filter_out_and_handle_revoked_sms_values` | internal rule | self, sms_values_all | `mass_mailing_sms`, `sms` |  | Meant to be overridden to filter out and handle sms that must not be sent.  :param dict sms_values_all: sms values by res_id :returns: filtered sms_vals_all :rtype: dict |
| `_get_blacklist_record_ids` | preparation rule | self, records, recipients_info | `sms` |  | Get a list of blacklisted records. Those will be directly canceled with the right error code. |
| `_get_optout_record_ids` | preparation rule | self, records, recipients_info | `mass_mailing_sms`, `sms` |  | Compute opt-outed contacts, not necessarily blacklisted. Void by default as no opt-out mechanism exist in SMS, see SMS Marketing. |
| `_get_done_record_ids` | preparation rule | self, records, recipients_info | `mass_mailing_sms`, `sms` |  | Get a list of already-done records. Order of record set is used to spot duplicates so pay attention to it if necessary. |
| `_prepare_recipient_values` | preparation rule | self, records | `sms` |  |  |
| `_prepare_body_values` | preparation rule | self, records | `mass_mailing_sms`, `sms` |  |  |
| `_prepare_mass_sms_values` | preparation rule | self, records | `mass_mailing_sms`, `sms_twilio`, `sms` |  |  |
| `_prepare_mass_sms` | preparation rule | self, records, sms_record_values | `mass_mailing_sms`, `sms` |  |  |
| `_prepare_log_body_values` | preparation rule | self, sms_records_values | `sms` |  |  |
| `_prepare_mass_log_values` | preparation rule | self, records, sms_records_values | `sms` |  |  |
| `_get_additional_render_context` | preparation rule | self | `sms` |  | Return a dict associating fields with their relevant render context if any.  e.g. {'body': {'additional_value': self.env.context.get('additional_value')}} |
| `_get_composer_values` | preparation rule | self, composition_mode, res_model, res_id, body, template_id | `sms` |  |  |
| `_get_records` | preparation rule | self | `sms` |  |  |
| `_get_unsubscribe_url` | preparation rule | self, mailing_id, trace_code | `mass_mailing_sms` |  |  |
| `_get_unsubscribe_info` | preparation rule | self, url | `mass_mailing_sms` | model |  |
| `_prepare_mass_sms_trace_values` | preparation rule | self, record, sms_values | `mass_mailing_sms` |  |  |
| `_is_mass_sms` | internal rule | self | `mass_mailing_sms` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_compute_sanitized_numbers` | UserError | Following numbers are not correctly encoded: %s | `sms` |
| `action_send_sms` | UserError | Invalid recipient number. Please update it. | `sms` |
| `action_send_sms` | UserError | %s invalid recipients | `sms` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `sms` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing_sms.sms_composer_view_form` | xpath | `sms.sms_composer_view_form` | `utm_campaign_id`, `mailing_id` |  |  | `mass_mailing_sms` |
| `sms.sms_composer_view_form` | form |  | `res_model_description`, `recipient_invalid_count`, `res_ids_count`, `composition_mode`, `comment_single_recipient`, `res_id`, `res_ids`, `res_model`, `mass_force_send`, `recipient_single_valid`, `recipient_single_number`, `number_field_name`, `numbers`, `sanitized_numbers`, `template_id`, `recipient_single_description`, `recipient_single_number_itf`, `body`, `body`, `body`, `mass_keep_log`, `use_exclusion_list` | `Send`, `Send`, `Put in queue`, `Send now`, `Discard` |  | `sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm_sms.crm_lead_act_window_sms_composer_single` | Send SMS | form |  | `{             'default_composition_mode': 'mass',             'default_mass_keep_log': True,             'default_res_ids': active_ids         }` | new | `crm_sms` |
| `crm_sms.crm_lead_act_window_sms_composer_multi` | Send SMS | form |  | `{             'default_composition_mode': 'comment',             'default_res_id': active_id,         }` | new | `crm_sms` |
| `project_sms.project_project_act_window_sms_composer` | Send SMS | form |  | `{             'default_composition_mode': 'mass',             'default_mass_keep_log': True,             'default_res_ids': active_ids,         }` | new | `project_sms` |
| `project_sms.project_task_act_window_sms_composer` | Send SMS | form |  | `{             'default_composition_mode': 'mass',             'default_mass_keep_log': True,             'default_res_ids': active_ids,         }` | new | `project_sms` |
| `sms.sms_composer_action_form` | Send SMS | form |  |  | new | `sms` |
| `sms.res_partner_act_window_sms_composer_multi` | Send SMS | form |  | `{             'default_composition_mode': 'mass',             'default_mass_keep_log': True,             'default_res_ids': active_ids         }` | new | `sms` |
| `sms.res_partner_act_window_sms_composer_single` | Send SMS | form |  | `{             'default_composition_mode': 'comment',             'default_res_id': active_id,         }` | new | `sms` |

Machine-readable definition: `../../../schemas/data/entities/sms.composer.json`; views: `../../../schemas/interfaces/views/sms.composer.json`.
