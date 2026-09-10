# Email composition wizard (`mail.compose.message`)

**Transport name:** `mail.compose.message`  
**Storage name:** `mail_compose_message`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mail`  
**Extended by packages:** `mass_mailing`, `marketing_card`

Description: Email composition wizard

## Identity and behavior

- Mixins (classical inheritance): `mail.composer.mixin`

## Fields (43)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `subject` | Subject | single line text |  | computed by rule `_compute_subject` and stored |
| `body` | Contents | rich text |  | computed by rule `_compute_body` and stored |
| `parent_id` | Parent Message | many to one | `mail.message` | on delete of the target: set null |
| `template_id` | Use template | many to one | `mail.template` | restricted by domain `[('model', '=', model), '\|', ('user_id','=', False), ('user_id', '=', uid)]` |
| `attachment_ids` | Attachments | many to many | `ir.attachment` | computed by rule `_compute_attachment_ids` and stored; association table `mail_compose_message_ir_attachments_rel` |
| `email_layout_xmlid` | Email Notification Layout | single line text |  | computed by rule `_compute_email_layout_xmlid` and stored; not copied on duplication |
| `email_add_signature` | Add signature | boolean |  | computed by rule `_compute_email_add_signature` and stored |
| `email_from` | From | single line text |  | computed by rule `_compute_authorship` and stored; Help: Email address of the sender. This field is set when no matching partner is found and replaces the author_id field in the chatter. |
| `author_id` | Author | many to one | `res.partner` | computed by rule `_compute_authorship` and stored; Help: Author of the message. If not set, email_from may hold an email address that did not match any partner. |
| `composition_mode` | Composition mode | selection |  | default `comment` |
| `composition_batch` | Batch composition | boolean |  | computed by rule `_compute_composition_batch` (not stored) |
| `composition_comment_option` | Comment Options | selection |  |  |
| `model` | Related Document Model | single line text |  | computed by rule `_compute_model` and stored |
| `model_is_thread` | Thread-Enabled | boolean |  | computed by rule `_compute_model_is_thread` (not stored) |
| `res_ids` | Related Document identifiers | multi line text |  | computed by rule `_compute_res_ids` and stored |
| `res_domain` | Active domain | multi line text |  |  |
| `res_domain_user_id` | Responsible | many to one | `res.users` | Help: Used as context used to evaluate composer domain |
| `record_alias_domain_id` | Alias Domain | many to one | `mail.alias.domain` | computed by rule `_compute_record_environment` and stored |
| `record_company_id` | Company | many to one | `res.company` | computed by rule `_compute_record_environment` and stored |
| `message_type` | Type | selection |  | required; default `comment`; Help: Message type: email for email message, notification for system message, comment for other messages such as user replies |
| `subtype_id` | Subtype | many to one | `mail.message.subtype` | computed by rule `_compute_subtype_id` and stored; on delete of the target: set null |
| `subtype_is_log` | Is a log | boolean |  | computed by rule `_compute_subtype_is_log` (not stored) |
| `mail_activity_type_id` | Mail Activity Type | many to one | `mail.activity.type` | on delete of the target: set null |
| `reply_to` | Reply To | single line text |  | computed by rule `_compute_reply_to` and stored; Help: Reply email address. Setting the reply_to bypasses the automatic thread creation. |
| `reply_to_force_new` | Considers answers as new thread | boolean |  | computed by rule `_compute_reply_to_force_new` and stored; Help: Manage answers as new incoming emails instead of replies going to the same thread. |
| `reply_to_mode` | Replies | selection |  | computed by rule `_compute_reply_to_mode` (not stored); writable through an inverse rule; Help: Original Discussion: Answers go in the original document discussion thread.   Another Email Address: Answers go to the email address mentioned in the tracking message-id instead of original document discussion thread.   This has an impact on the generated message-id. |
| `partner_ids` | Additional Contacts | many to many | `res.partner` | computed by rule `_compute_partner_ids` and stored; association table `mail_compose_message_res_partner_rel` |
| `partner_ids_all_have_email` | Partner Identifiers All Have Email | boolean |  | computed by rule `_compute_partner_ids_all_have_email` (not stored) |
| `notified_bcc_contains_share` | Is an external partner follower of the document? | boolean |  | computed by rule `_compute_notified_bcc_contains_share` (not stored) |
| `auto_delete` | Delete Emails | boolean |  | computed by rule `_compute_auto_delete` and stored; Help: This option permanently removes any track of email after it's been sent, including from the Technical menu in the Settings, in order to preserve storage space of your Odoo database. |
| `auto_delete_keep_log` | Keep Message Copy | boolean |  | computed by rule `_compute_auto_delete_keep_log` and stored; Help: Keep a copy of the email content if emails are removed (mass mailing only) |
| `force_send` | Send mailing or notifications directly | boolean |  | computed by rule `_compute_force_send` and stored |
| `mail_server_id` | Outgoing mail server | many to one | `ir.mail_server` | computed by rule `_compute_mail_server_id` and stored |
| `notify_author` | Notify Author | boolean |  | computed by rule `_compute_notify_author` and stored |
| `notify_author_mention` | Notify Author Mention | boolean |  | computed by rule `_compute_notify_author_mention` and stored |
| `notify_skip_followers` | Notify Skip Followers | boolean |  | computed by rule `_compute_notify_skip_followers` and stored |
| `scheduled_date` | Scheduled Date | single line text |  | computed by rule `_compute_scheduled_date` and stored; Help: In comment mode: if set, postpone notifications sending. In mass mail mode: if sent, send emails after that date. This date is considered as being in UTC timezone. |
| `use_exclusion_list` | Use Exclusion List | boolean |  | default `True`; not copied on duplication; Help: Prevent sending messages to blacklisted contacts. Disable only when absolutely necessary. |
| `template_name` | Template Name | single line text |  |  |
| `mass_mailing_id` | Mass Mailing | many to one | `mailing.mailing` | on delete of the target: cascade |
| `campaign_id` | Mass Mailing Campaign | many to one | `utm.campaign` | on delete of the target: set null |
| `mass_mailing_name` | Mass Mailing Name | single line text |  | Help: If set, a mass mailing will be created so that you can track its results in the Email Marketing app. |
| `mailing_list_ids` | Mailing List | many to many | `mailing.list` |  |

## Selection values

### `composition_mode` (Composition mode)

| Value | Label |
|---|---|
| `comment` | Post on a document |
| `mass_mail` | Email Mass Mailing |

### `composition_comment_option` (Comment Options)

| Value | Label |
|---|---|
| `reply_all` | Reply-All |
| `forward` | Forward |

### `message_type` (Type)

| Value | Label |
|---|---|
| `auto_comment` | Automated Targeted Notification |
| `comment` | Comment |
| `notification` | System notification |

### `reply_to_mode` (Replies)

| Value | Label |
|---|---|
| `update` | Store email and replies in the chatter of each record |
| `new` | Collect replies on a specific email address |

## Operations (66)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mail` | model | Handle composition mode and contextual computation, until moving to computed fields. Support active_model / active_id(s) as valid default values, as this comes from standard web client usage.  Note that supporting active_ids through composer is still done, as we may have to give a huge list of IDs that won't fit into res_ids field. |
| `_check_res_ids` | validation | self | `mail` | constrains: `res_ids` | Check res_ids is a valid list of integers (or Falsy). |
| `_check_res_domain` | validation | self | `mail` | constrains: `res_domain` | Check domain is a valid domain if set (otherwise it is considered as a Falsy leaf. |
| `_compute_subject` | computation | self | `mail` | depends: `composition_mode`, `model`, `parent_id`, `res_domain`, `res_ids`, `template_id` | Computation is coming either form template, either from context. When having a template with a value set, copy it (in batch mode) or render it (in monorecord comment mode) on the composer. Otherwise it comes from the parent (if set), or computed based on the generic '_message_compute_subject' method or to the record display_name in monorecord comment mode, or set to False. When removing the template, reset it. |
| `_compute_body` | computation | self | `mail` | depends: `composition_mode`, `model`, `res_domain`, `res_ids`, `template_id` | Computation is coming either from template, either reset. When having a template with a value set, copy it (in batch mode) or render it (in monorecord comment mode) on the composer. When removing the template, reset it. |
| `_compute_attachment_ids` | computation | self | `mail` | depends: `composition_mode`, `model`, `res_domain`, `res_ids`, `template_id` | Computation is based on template and composition mode. In monorecord comment mode, template is used to generate attachments based on both attachment_ids of template, and reports coming from report_template_ids. Those are generated based on the current record to display. As template generation returns a list of tuples, new attachments are created on the fly during the compute.  In batch or email mode, only attachment_ids from template are used on the composer. Reports will be generated at sending time.  When template is removed, attachments are reset. |
| `_compute_email_add_signature` | computation | self | `mail` | depends: `template_id` | When having a template, consider it defines completely body and do not add signature. Without template, add signature by default in comment due to post processing for notification emails. Mailing mode does not handle signature. |
| `_compute_email_layout_xmlid` | computation | self | `mail` | depends: `template_id` | Computation is coming either from template, either reset. When having a template with a value set, set it on composer.When removing the template, reset it. |
| `_compute_authorship` | computation | self | `mail` | depends: `composition_mode`, `email_from`, `model`, `res_domain`, `res_ids`, `template_id` | Computation is coming either from template, either from context. When having a template with a value set, copy it (in batch mode) or render it (in monorecord comment mode) on the composer. Otherwise try to take current user's email. When removing the template, fallback on default thread behavior (which is current user's email).  Author is not controllable from the template currently. We therefore try to synchronize it with the given email_from (in rendered mode to avoid trying to find partner based on qweb expressions), or fallback on current user. |
| `_compute_composition_batch` | computation | self | `mail` | depends: `res_domain`, `res_ids` | Determine if batch mode is activated:  * using res_domain: always batch (even if result is singleton at a   given time, it is user and time dependent, hence batch); * res_ids: if more than one item in the list (void and singleton are   not batch); |
| `_compute_model` | computation | self | `mail` | depends: `composition_mode`, `parent_id` | Model can be set from parent or using 'active_model' context key frequently used as the composer is most invoked from list or form views. |
| `_compute_model_is_thread` | computation | self | `mail` | depends: `model` | Determine if model is thread enabled. |
| `_compute_res_ids` | computation | self | `mail` | depends: `composition_mode`, `parent_id` | Computation may come from parent in comment mode, if set. It takes the parent message's res_id. Otherwise the composer uses the 'active_ids' context key, unless it is too big to be stored in database. Indeed when invoked for big mailings, 'active_ids' may be a very big list. Support of 'active_ids' when sending is granted in order to not always rely on 'res_ids' field. When 'active_ids' is not present, fallback on 'active_id'. |
| `_compute_record_environment` | computation | self | `mail` | depends: `composition_mode`, `model`, `res_domain`, `res_ids` | In monorecord mode, fetch record company and the linked alias domain, easing future processing notably at post and notification sending time.  In batch mode it makes no sense to compute a single company, it will be dynamically generated. |
| `_compute_subtype_id` | computation | self | `mail` | depends: `composition_mode` | Computation defaults from composition mode. Subtype is not used in mass mail mode, and is comment for comment mode. |
| `_compute_subtype_is_log` | computation | self | `mail` | depends: `subtype_id` | In comment mode, tells whether the subtype is a note. Subtype has no use in email mode, and this field will be False. |
| `_compute_reply_to` | computation | self | `mail` | depends: `composition_mode`, `model`, `res_domain`, `res_ids`, `template_id` | Computation is coming either from template, either reset. When having a template with a value set, copy it (in batch mode) or render it (in monorecord comment mode) on the composer. When removing the template, reset it. |
| `_compute_reply_to_force_new` | computation | self | `mail` | depends: `model`, `reply_to` | If model does not inherit from MailThread, avoid replies to be considered as thread updates, they will instead follow the routing rules (alias, ...). Other models by default collect replies in the same thread, unless a reply_to is forced, usually either throuh a template, either because of mailing mode. |
| `_compute_reply_to_mode` | computation | self | `mail` | depends: `reply_to_force_new` |  |
| `_inverse_reply_to_mode` | inverse computation | self | `mail` |  |  |
| `_compute_partner_ids` | computation | self | `mail` | depends: `composition_mode`, `model`, `parent_id`, `res_domain`, `res_ids`, `subtype_id`, `template_id` | Computation is coming either from template, either from context. When having a template it uses its 3 fields 'email_cc', 'email_to' and 'partner_to', in monorecord comment mode. Emails are converted into partners, creating new ones when the email does not match any existing partner. Composer does not deal with emails but only with partners. When having a template in other modes, no recipients are computed as it is done at sending time. When removing the template, reset it.  When not having a template, recipients may come from the parent in comment mode, to be sure to notify the same people. |
| `_compute_partner_ids_all_have_email` | computation | self | `mail` | depends: `partner_ids` |  |
| `_compute_notified_bcc_contains_share` | computation | self | `mail` | depends: `composition_batch`, `composition_mode`, `message_type`, `model`, `res_ids`, `subtype_id` | When being in monorecord comment mode, compute 'bcc' which are followers that are going to be 'silently' notified by the message. |
| `_compute_auto_delete` | computation | self | `mail` | depends: `composition_mode`, `template_id` | Computation is coming either from template, either from composition mode. When having a template, its value is copied. Without template it is True in comment mode to remove notification emails by default. In email mode we keep emails (backward compatibility mode). |
| `_compute_auto_delete_keep_log` | computation | self | `mail` | depends: `composition_mode`, `auto_delete` | Keep logs is used only in email mode. It is used to keep the core message when unlinking sent emails. It allows to keep the message as a trace in the record's chatter. In other modes it has no use and can be set to False. When auto_delete is turned off it has no usage. |
| `_compute_force_send` | computation | self | `mail` | depends: `composition_mode`, `model`, `res_domain`, `res_ids` | When being in single record mode, we force_send (post on a record or send a single email right away). In batch mode: comment always uses the email queue (lot of potentially different emails to craft). Mass mailing mode depends on number of recipients and is configurable using 'mail.mail.force.send.limit' configuration parameter (default=100). Using a domain forces the email queue usage as it depends on actual evaluation and is generally used for big batches anyway. |
| `_compute_mail_server_id` | computation | self | `mail` | depends: `template_id` | Copy value from template when updating it, if set on template. When removing the template, reset it. |
| `_compute_notify_author` | computation | self | `mail` | depends: `composition_mode` | Used only in 'comment' mode, controls 'notify_author' notification parameter |
| `_compute_notify_author_mention` | computation | self | `mail` | depends: `composition_mode` | Used only in 'comment' mode, controls 'notify_author_mention' notification parameter. |
| `_compute_notify_skip_followers` | computation | self | `mail` | depends: `composition_mode`, `composition_comment_option` | Used only in 'comment' mode, controls 'notify_skip_followers' notification parameter. 'Reply-All' behavior triggers skipping followers. |
| `_compute_scheduled_date` | computation | self | `mail` | depends: `composition_mode`, `model`, `res_ids`, `template_id` | Computation is coming either from template, either reset. When having a template with a value set, copy it (in batch mode) or render it (in monorecord comment mode) on the composer. When removing the template, reset it. |
| `_compute_lang` | computation | self | `mail` | depends: `template_id` | Computation is coming either from template, either reset. When having a template with a value set, copy it (in batch mode) or render it (in monorecord comment mode) on the composer. When removing the template, reset it. |
| `_compute_render_model` | computation | self | `mail` | depends: `model` |  |
| `_compute_can_edit_body` | computation | self | `mail` |  | Can edit the body if we are not in "mass_mail" mode because the template is rendered before it's modified. |
| `_compute_field_value` | computation | self, field | `mail` |  |  |
| `_gc_lost_attachments` | background operation | self | `mail` | autovacuum | Garbage collect lost mail attachments. Those are attachments - linked to res_model 'mail.compose.message', the composer wizard - with res_id 0, because they were created outside of an existing     wizard (typically user input through Chatter or reports     created on-the-fly by the templates) - unused since at least one day (create_date and write_date) |
| `action_schedule_message` | user action | self | `mail` |  |  |
| `_prepare_schedule_message_post_values` | preparation rule | self, post_values | `mail` |  | Override this method to add additional values to the 'mail.scheduled.message' record creation. This is useful for custom modules that need to add specific fields to the scheduled message. :param post_values: dict of post values to be used for the scheduled message :param wizard: mail.compose.message record that is used to schedule the message :return: dict of additional values to be added to the scheduled message creation |
| `_action_schedule_message` | internal rule | self | `mail` |  | Create a 'scheduled message' to be posted automatically later. |
| `action_send_mail` | user action | self | `mail` |  | Used for action button that do not accept arguments. |
| `_action_send_mail` | internal rule | self, auto_commit | `mail`, `mass_mailing` |  | Process the wizard content and proceed with sending the related     email(s), rendering any template patterns on the fly if needed.  :return: (     result_mails_su: in mass mode, sent emails (as sudo),     result_messages: in comment mode, posted messages ) |
| `_action_send_mail_comment` | internal rule | self, res_ids | `mail` |  | Send in comment mode. It calls message_post on model, or the generic implementation of it if not available (as message_notify). |
| `_action_send_mail_mass_mail` | internal rule | self, res_ids, auto_commit | `mail` |  | Send in mass mail mode. Mails are sudo-ed, as when going through _prepare_mail_values standard access rights on related records will be checked when browsing them to compute mail values. If people have access to the records they have rights to create lots of emails in sudo as it is considered as a technical model. |
| `_generate_mail_notification_values` | internal rule | self, mails | `mail`, `mass_mailing` |  | Prevent notification creation as traces are generated. |
| `open_template_creation_wizard` | operation | self | `mail` |  | hit save as template button: opens a wizard that prompts for the template's subject. `create_mail_template` is called when saving the new wizard. |
| `create_mail_template` | operation | self | `mail` |  | creates a mail template with the current mail composer's fields |
| `cancel_save_template` | operation | self | `mail` |  | Restore old subject when canceling the 'save as template' action as it was erased to let user give a more custom input. |
| `_invalid_email_state` | internal rule | self | `mail`, `mass_mailing` |  | Gives state of an email when the address is invalid or missing.  We consider that if not keeping logs, users will not care to correct record-wise errors as it was "send and forget". Whereas if they do keep logs, they will want to know that the message was not actually sent. |
| `_prepare_mail_values` | preparation rule | self, res_ids | `mail`, `mass_mailing` |  | Generate the values that will be used by send_mail to create either  mail_messages or mail_mails depending on composition mode.  Some summarized information on generation: mail versus message fields (or both), and static (never rendered) versus dynamic (raw or rendered).  MAIL     STA - 'auto_delete',     DYN - 'body_html',     STA - 'force_send',  (notify parameter)     STA - 'model',     DYN - 'recipient_ids',  (from partner_ids)     DYN - 'res_id',     STA - 'is_notification',  MESSAGE     DYN - 'body',     STA - 'email_add_signature',     STA - 'email_layout_xmlid',     DYN - 'force_email_ |
| `_manage_mail_values` | internal rule | self, mail_values_all | `mail`, `mass_mailing` |  | Meant to be overridden to filter out and handle mail that must not be sent.  :param dict mail_values_all: mail values by res_id :return: filtered mail_vals_all :rtype: dict |
| `_prepare_mail_values_static` | preparation rule | self | `mail` |  | Prepare values always valid, not rendered or dynamic whatever the composition mode and related records.  :returns: a dict of (field name, value) to be used to populate   values for each res_id in '_prepare_mail_values'; :rtype: dict |
| `_prepare_mail_values_dynamic` | preparation rule | self, res_ids | `mail`, `marketing_card` |  | Generate values based on composer content as well as its template based on records given by res_ids.  Part of the advanced rendering is delegated to template, notably recipients or attachments dynamic generation. See sub methods for more details.  :param list res_ids: list of record IDs on which composer runs;  :returns: for each res_id, the generated values used to   populate in '_prepare_mail_values'; :rtype: dict |
| `_prepare_mail_values_rendered` | preparation rule | self, res_ids | `mail` |  | Generate values that are already rendered. This is used mainly in monorecord mode, when the wizard contains value already generated (e.g. "Send by email" on a sale order, in form view).  :param list res_ids: list of record IDs on which composer runs;  :returns: for each res_id, the generated values used to   populate in '_prepare_mail_values'; :rtype: dict |
| `_process_mail_values_state` | background operation | self, mail_values_dict | `mail` |  | When being in mass mailing, avoid sending emails to void or invalid emails. For that purpose a processing of generated values allows to give a state and a failure type to mail.mail records that will be created at sending time.  :param dict mail_values_dict: as generated by '_prepare_mail_values';  :return: updated mail_values_dict |
| `_generate_template_for_composer` | internal rule | self, res_ids, render_fields, allow_suggested, find_or_create_partners | `mail` |  | Generate values based on template and relevant values for the mail.compose.message wizard.  :param list res_ids: list of record IDs on which template is rendered; :param list render_fields: list of fields to render on template; :param boolean allow_suggested: when computing default recipients,   include suggested recipients in addition to minimal defaults   (see ``Template._generate_template_recipients``); :param boolean find_or_create_partners: transform emails into partners   (see ``Template._generate_template_recipients``);  :returns: a dict containing all asked fields for each record ID gi |
| `_get_blacklist_record_ids` | preparation rule | self, mail_values_dict, recipients_info | `mail` |  |  |
| `_get_done_emails` | preparation rule | self, mail_values_dict | `mail`, `marketing_card`, `mass_mailing` |  | Consider every target gets a different card, hence we don't want unique message per email address. |
| `_get_optout_emails` | preparation rule | self, mail_values_dict | `mail`, `mass_mailing` |  |  |
| `_get_recipients_data` | preparation rule | self, mail_values_dict | `mail` |  |  |
| `_evaluate_res_domain` | internal rule | self | `mail` |  | Parse composer domain, which can be: an already valid list or tuple (generally in code), a list or tuple as a string (coming from actions). Void strings are considered as a falsy domain.  :return: an Odoo domain (list of leaves) |
| `_evaluate_res_ids` | internal rule | self | `mail` |  | Parse composer res_ids, which can be: an already valid list or tuple (generally in code), a list or tuple as a string (coming from actions). Void strings / missing values are evaluated as an empty list.  Note that 'active_ids' context key is supported at this point as mailing on big ID list would create issues if stored in database.  Another context key 'composer_force_res_ids' is temporarily supported to ease support of accounting wizard, while waiting to implement a proper solution to language management.  :return: a list of IDs (empty list in case of falsy strings) |
| `_set_value_from_template` | internal rule | self, template_fname, composer_fname | `mail` |  | Set composer value from its template counterpart. In monorecord comment mode, we get directly the rendered value, giving the real value to the user. Otherwise we get the raw (unrendered) value from template, as it will be rendered at send time (for mass mail, whatever the number of contextual records to mail) or before posting on records (for comment in batch).  :param str template_fname: name of field on template model, used to   fetch the value (and maybe render it); :param str composer_fname: name of field on composer model, when field   names do not match (e.g. body_html on template used t |
| `_prepare_mail_values_mailing_traces` | preparation rule | self, mail_values_all | `mass_mailing` |  |  |
| `_prepare_mailing_values` | preparation rule | self | `mass_mailing` |  |  |
| `_is_mass_mailing` | internal rule | self | `mass_mailing` |  |  |
| `_process_generic_card_url_body` | background operation | self, card_body_pairs | `marketing_card` | model | Update the bodies with the specific card url for that res_id and create a card.  example: (1, "/cards/9/preview") -> (1, "/cards/9/1/abchashtoken/preview") + new card as side-effect  :return: processed bodies in the order they were received |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_action_schedule_message` | UserError | A message can only be scheduled in monocomment mode | `mail` |
| `_action_schedule_message` | UserError | A scheduled date is needed to schedule a message | `mail` |
| `_action_send_mail_comment` | UserError | No recipient found. | `mail` |
| `create_mail_template` | UserError | Template creation from composer requires a valid model. | `mail` |
| `_evaluate_res_domain` | ValidationError | Invalid domain “%(domain)s” (type “%(domain_type)s”) | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Mail Compose Message Rule | global (all users) | `[('create_uid', '=', user.id)]` | True | True | False | False |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.email_compose_message_wizard_form` | form |  | `author_id`, `auto_delete`, `auto_delete_keep_log`, `composition_batch`, `composition_comment_option`, `composition_mode`, `email_layout_xmlid`, `force_send`, `lang`, `mail_server_id`, `model`, `model_is_thread`, `notified_bcc_contains_share`, `notify_author`, `notify_author_mention`, `notify_skip_followers`, `parent_id`, `partner_ids_all_have_email`, `record_alias_domain_id`, `record_company_id`, `render_model`, `res_domain`, `res_domain_user_id`, `res_ids`, `scheduled_date`, `subtype_id`, `subtype_is_log`, `use_exclusion_list`, `email_from`, `partner_ids`, `partner_ids`, `subject`, `reply_to`, `can_edit_body`, `body`, `attachment_ids`, `body`, `attachment_ids`, `reply_to_force_new`, `reply_to_mode`, `use_exclusion_list`, `attachment_ids`, `template_id`, `scheduled_date` | `Send`, `Log`, `Schedule`, `Send`, `Log`, `Schedule`, `Discard` |  | `mail` |
| `mail.mail_compose_message_view_form_template_save` | form |  | `template_name`, `model` | `Save Template`, `Discard` |  | `mail` |
| `mass_mailing.email_compose_form_mass_mailing` | xpath | `mail.email_compose_message_wizard_form` | `campaign_id`, `mass_mailing_name` |  |  | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.account_send_payment_receipt_by_email_action` | Send receipt by email | form |  | `{                 'mail_post_autofollow': True,                 'default_composition_mode': 'comment',                 'default_template_id': ref('account.mail_template_data_payment_receipt'),                 'default_email_layout_xmlid': 'mail.mail_notification_light',             }` | new | `account` |
| `account.account_send_payment_receipt_by_email_action_multi` | Send receipts by email | form |  | `{                 'mail_post_autofollow': True,                 'default_composition_mode': 'mass_mail',                 'default_template_id': ref('account.mail_template_data_payment_receipt'),                 'default_email_layout_xmlid': 'mail.mail_notification_light',             }` | new | `account` |
| `crm.action_lead_mail_compose` | Send email | form |  | `{     'default_composition_mode': 'comment',                 }` | new | `crm` |
| `crm.action_lead_mass_mail` | Send email | form |  | `{     'default_composition_mode': 'mass_mail',                 }` | new | `crm` |
| `mail.action_email_compose_message_wizard` | Compose Email | form |  |  | new | `mail` |
| `mail.action_partner_mass_mail` | Send email | form |  | `{                 'default_composition_mode': 'mass_mail',                 'default_partner_to': '{{ object.id or \'\' }}',                 'default_subtype_xmlid': 'mail.mt_comment',                 'default_reply_to_force_new': True,             }` | new | `mail` |
| `mail_group.mail_compose_message_action_mail_group` | Send email | form |  | `{             'default_composition_mode': 'mass_mail',             'default_model': 'mail.group.member',             'default_res_ids': active_ids         }` | new | `mail_group` |
| `project.action_send_mail_project_project` | Send Email | form |  | `{                 'default_composition_mode': 'mass_mail',             }` | new | `project` |
| `project.action_send_mail_project_task` | Send Email | form |  | `{                 'default_composition_mode': 'mass_mail',             }` | new | `project` |
| `stock.action_lead_mass_mail` | Send email | form |  | `{                 'default_composition_mode': 'mass_mail',             }` | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/mail.compose.message.json`; views: `../../../schemas/interfaces/views/mail.compose.message.json`.
