# Email Templates (`mail.template`)

**Transport name:** `mail.template`  
**Storage name:** `mail_template`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `account`, `event`, `pos_self_order`

Description: Email Templates

## Identity and behavior

- Mixins (classical inheritance): `mail.render.mixin`, `template.reset.mixin`, `pos.load.mixin`
- Default ordering: `user_id, name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (26)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | translatable |
| `description` | Template Description | multi line text |  | translatable; Help: This field is used for internal description of the template's usage. |
| `active` | Active | boolean |  | default `True` |
| `template_category` | Template Category | selection |  | computed by rule `_compute_template_category` (not stored); searchable through a search rule |
| `model_id` | Applies to | many to one | `ir.model` | on delete of the target: cascade; restricted by domain `_get_non_abstract_models_domain` |
| `model` | Related Document Model | single line text |  | read only; related through path `model_id.model` and stored; indexed |
| `subject` | Subject | single line text |  | translatable; Help: Subject (placeholders may be used here) |
| `email_from` | Send From | single line text |  | Help: Sender address (placeholders may be used here). If not set, the default value will be the author's email alias if configured, or email address. |
| `user_id` | Owner | many to one | `res.users` | restricted by domain `[('share', '=', False)]` |
| `use_default_to` | Default Recipients | boolean |  | default `True`; Help: Default recipients of the record: - partner (using id on a partner or the partner_id field) OR - email (using email_from or email field) |
| `email_to` | To (Emails) | single line text |  | Help: Comma-separated recipient addresses (placeholders may be used here) |
| `partner_to` | To (Partners) | single line text |  | Help: Comma-separated ids of recipient partners (placeholders may be used here) |
| `email_cc` | Cc | single line text |  | Help: Carbon copy recipients (placeholders may be used here) |
| `reply_to` | Reply To | single line text |  | Help: Email address to which replies will be redirected when sending emails in mass; only used when the reply is not logged in the original discussion thread. |
| `body_html` | Body | rich text |  | translatable |
| `attachment_ids` | Attachments | many to many | `ir.attachment` | association table `email_template_attachment_rel` |
| `report_template_ids` | Dynamic Reports | many to many | `ir.actions.report` | restricted by domain `[('model', '=', model)]`; association table `mail_template_ir_actions_report_rel` |
| `email_layout_xmlid` | Email Notification Layout | single line text |  | not copied on duplication |
| `mail_server_id` | Outgoing Mail Server | many to one | `ir.mail_server` | indexed (btree_not_null); Help: Optional preferred server for outgoing mails. If not set, the highest priority one will be used. |
| `scheduled_date` | Scheduled Date | single line text |  | Help: If set, the queue manager will send the email after the date. If not set, the email will be send as soon as possible. You can use dynamic expression. |
| `auto_delete` | Auto Delete | boolean |  | default `True`; Help: This option permanently removes any track of email after it's been sent, including from the Technical menu in the Settings, in order to preserve storage space of your Odoo database. |
| `ref_ir_act_window` | Sidebar action | many to one | `ir.actions.act_window` | read only; not copied on duplication; Help: Sidebar action to make this template available on records of the related document model |
| `can_write` | Can Write | boolean |  | computed by rule `_compute_can_write` (not stored); Help: The current user can edit the template. |
| `is_template_editor` | Is Template Editor | boolean |  | computed by rule `_compute_is_template_editor` (not stored) |
| `has_dynamic_reports` | Has Dynamic Reports | boolean |  | computed by rule `_compute_has_dynamic_reports` (not stored) |
| `has_mail_server` | Has Mail Server | boolean |  | computed by rule `_compute_has_mail_server` (not stored) |

## Selection values

### `template_category` (Template Category)

| Value | Label |
|---|---|
| `base_template` | Base Template |
| `hidden_template` | Hidden Template |
| `custom_template` | Custom Template |

## Operations (38)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mail` | model |  |
| `_get_non_abstract_models_domain` | preparation rule | self | `mail` |  |  |
| `_compute_has_dynamic_reports` | computation | self | `mail` | depends: `model` |  |
| `_compute_has_mail_server` | computation | self | `mail` |  |  |
| `_compute_render_model` | computation | self | `mail` | depends: `model` |  |
| `_compute_can_write` | computation | self | `mail` | depends_context: `uid` |  |
| `_compute_is_template_editor` | computation | self | `mail` | depends_context: `uid` |  |
| `_compute_template_category` | computation | self | `mail` | depends: `active`, `description` | Base templates (or master templates) are active templates having a description and an XML ID. User defined templates (no xml id), templates without description or archived templates are not base templates anymore. |
| `_search_template_category` | search rule | self, operator, value | `mail` | model |  |
| `_onchange_model` | on change | self | `mail` | onchange: `model` |  |
| `_fix_attachment_ownership` | internal rule | self | `mail` |  |  |
| `_check_abstract_models` | validation | self, vals_list | `mail` |  |  |
| `_check_can_be_rendered` | validation | self, fnames, render_options | `mail` |  |  |
| `_get_dynamic_field_names` | preparation rule | self | `mail` |  |  |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `unlink` | lifecycle override | self | `event`, `mail` |  |  |
| `copy_data` | lifecycle override | self, default | `mail` |  |  |
| `copy` | lifecycle override | self, default | `mail` |  |  |
| `unlink_action` | operation | self | `mail` |  |  |
| `create_action` | operation | self | `mail` |  |  |
| `action_open_mail_preview` | user action | self | `mail` |  |  |
| `_generate_template_attachments` | internal rule | self, res_ids, render_fields, render_results | `mail` |  | Render attachments of template 'self', returning values for records given by 'res_ids'. Note that ``report_template_ids`` returns values for 'attachments', as we have a list of tuple (report_name, base64 value) for those reports. It is considered as being the job of callers to transform those attachments into valid ``ir.attachment`` records.  :param list res_ids: list of record IDs on which template is rendered; :param list render_fields: list of fields to render on template which   are specific to attachments, e.g. attachment_ids or report_template_ids; :param dict render_results: res_ids-bas |
| `_generate_template_recipients` | internal rule | self, res_ids, render_fields, allow_suggested, find_or_create_partners, render_results | `mail` |  | Render recipients of the template 'self', returning values for records given by 'res_ids'. Default values can be generated instead of the template values if requested by template (see 'use_default_to' field). Email fields ('email_cc', 'email_to') are transformed into partners if requested (finding or creating partners). 'partner_to' field is transformed into 'partner_ids' field.  Note: for performance reason, information from records are transferred to created partners no matter the company. For example, if we have a record of company A and one of B with the same email and no related partner,  |
| `_generate_template_scheduled_date` | internal rule | self, res_ids, render_results | `mail` |  | Render scheduled date based on template 'self'. Specific parsing is done to ensure value matches ORM expected value: UTC but without timezone set in value.  :param list res_ids: list of record IDs on which template is rendered; :param dict render_results: res_ids-based dictionary of render values.   For each res_id, a dict of values based on render_fields is given;  :return: updated (or new) render_results; |
| `_generate_template_static_values` | internal rule | self, res_ids, render_fields, render_results | `mail` |  | Return values based on template 'self'. Those are not rendered nor dynamic, just static values used for configuration of emails.  :param list res_ids: list of record IDs on which template is rendered; :param list render_fields: list of fields to render, currently limited   to a subset (i.e. auto_delete, mail_server_id, model, res_id); :param dict render_results: res_ids-based dictionary of render values.   For each res_id, a dict of values based on render_fields is given;  :return: updated (or new) render_results; |
| `_generate_template` | internal rule | self, res_ids, render_fields, recipients_allow_suggested, find_or_create_partners | `mail` |  | Render values from template 'self' on records given by 'res_ids'. Those values are generally used to create a mail.mail or a mail.message. Model of records is the one defined on template.  :param list res_ids: list of record IDs on which template is rendered; :param list render_fields: list of fields to render on template;  # recipients generation :param boolean recipients_allow_suggested: when computing default   recipients, include suggested recipients in addition to minimal   defaults; :param boolean find_or_create_partners: transform emails into partners   (see ``_generate_template_recipie |
| `_parse_partner_to` | internal rule | cls, partner_to | `mail` |  |  |
| `_send_check_access` | internal rule | self, res_ids | `mail` |  |  |
| `send_mail` | operation | self, res_id, force_send, raise_exception, email_values, email_layout_xmlid | `mail` |  | Generates a new mail.mail. Template is rendered on record given by res_id and model coming from template.  :param int res_id: id of the record to render the template :param bool force_send: send email immediately; otherwise use the mail     queue (recommended); :param dict email_values: update generated mail with those values to further     customize the mail; :param str email_layout_xmlid: optional notification layout to encapsulate the     generated email; :returns: id of the mail.mail that was created |
| `send_mail_batch` | operation | self, res_ids, force_send, raise_exception, email_values, email_layout_xmlid | `mail` |  | Generates new mail.mails. Batch version of 'send_mail'.'  :param list res_ids: IDs of modelrecords on which template will be rendered  :returns: newly created mail.mail |
| `_has_unsafe_expression_template_qweb` | internal rule | self, source, model, fname | `mail` |  |  |
| `_has_unsafe_expression_template_inline_template` | internal rule | self, source, model, fname | `mail` |  |  |
| `_expression_is_default` | internal rule | self, source, model, fname | `mail` |  |  |
| `_unlink_except_master_mail_template` | internal rule | self | `account` | ondelete |  |
| `_search` | search rule | self, domain, *args, **kwargs | `event` | model | Context-based hack to filter reference field in a m2o search box to emulate a domain the ORM currently does not support.  As we can not specify a domain on a reference field, we added a context key `filter_template_on_event` on the template reference field. If this key is set, we add our domain in the `domain` in the `_search` method to filtrate the mail templates. |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_self_order` | model |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_abstract_models` | ValidationError | You may not define a template on an abstract model: %s | `mail` |
| `_check_can_be_rendered` | ValidationError | Oops! We couldn't save your template due to an issue.  Error: %(error_details)s  Correct it and try again. | `mail` |
| `_generate_template_attachments` | UserError | Unsupported report type %s found. | `mail` |
| `_unlink_except_master_mail_template` | UserError | You cannot delete this mail template, it is used in the invoice sending flow. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `mail` |
| `mail.group_mail_template_editor` | yes | yes | yes | yes | `mail` |
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Employees can only modify templates they have created or been assigned | `[Command.link(ref('base.group_user'))]` | `['\|', ('create_uid', '=', user.id), ('user_id', '=', user.id)]` | False | True | True | True |
| Mail Template Editors - Edit All Templates | `[Command.link(ref('group_mail_template_editor')), Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | False | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.email_template_form` | form |  | `ref_ir_act_window`, `template_fs`, `is_template_editor`, `name`, `model_id`, `model_id`, `subject`, `model`, `can_write`, `body_html`, `email_from`, `use_default_to`, `email_to`, `partner_to`, `email_cc`, `reply_to`, `lang`, `has_mail_server`, `mail_server_id`, `auto_delete`, `scheduled_date`, `has_dynamic_reports`, `attachment_ids`, `report_template_ids`, `user_id`, `description` | `Preview`, `Reset Template`, `Add Context Action`, `Remove Context Action` |  | `mail` |
| `mail.email_template_tree` | list |  | `mail_server_id`, `name`, `model_id`, `user_id`, `description`, `subject`, `email_from`, `email_to`, `partner_to` |  |  | `mail` |
| `mail.view_email_template_search` | search |  | `name`, `lang`, `model`, `model_id` |  | `My Templates`, `Base Templates`, `Custom Templates`, `SMTP Server`, `Model` | `mail` |
| `product_email_template.email_template_form_simplified` | form |  | `subject`, `name`, `model`, `body_html`, `attachment_ids` |  |  | `product_email_template` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.action_email_template_tree_all` | Email Templates | form,list |  | `{'search_default_base_templates': 1}` |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.template.json`; views: `../../../schemas/interfaces/views/mail.template.json`.
