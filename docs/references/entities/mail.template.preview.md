# Email Template Preview (`mail.template.preview`)

**Transport name:** `mail.template.preview`  
**Storage name:** `mail_template_preview`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mail`

Description: Email Template Preview

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mail_template_id` | Related Mail Template | many to one | `mail.template` | required |
| `model_id` | Targeted model | many to one | `ir.model` | related through path `mail_template_id.model_id` |
| `resource_ref` | Record | reference |  | computed by rule `_compute_resource_ref` and stored; values provided by rule `_selection_target_model` |
| `lang` | Template Preview Language | selection |  |  |
| `no_record` | No Record | boolean |  | computed by rule `_compute_no_record` (not stored) |
| `error_msg` | Error Message | single line text |  | computed by rule `_compute_mail_template_fields` (not stored) |
| `subject` | Subject | single line text |  | computed by rule `_compute_mail_template_fields` (not stored) |
| `email_from` | From | single line text |  | computed by rule `_compute_mail_template_fields` (not stored); Help: Sender address |
| `email_to` | To | single line text |  | computed by rule `_compute_mail_template_fields` (not stored); Help: Comma-separated recipient addresses |
| `email_cc` | Cc | single line text |  | computed by rule `_compute_mail_template_fields` (not stored); Help: Carbon copy recipients |
| `reply_to` | Reply-To | single line text |  | computed by rule `_compute_mail_template_fields` (not stored); Help: Preferred response address |
| `scheduled_date` | Scheduled Date | single line text |  | computed by rule `_compute_mail_template_fields` (not stored); Help: The queue manager will send the email after the date |
| `body_html` | Body | rich text |  | computed by rule `_compute_mail_template_fields` (not stored) |
| `attachment_ids` | Attachments | many to many | `ir.attachment` | computed by rule `_compute_mail_template_fields` (not stored) |
| `has_attachments` | Has Attachments | boolean |  | computed by rule `_compute_has_attachments` (not stored) |
| `has_several_languages_installed` | Has Several Languages Installed | boolean |  | computed by rule `_compute_has_several_languages_installed` (not stored) |
| `partner_ids` | Recipients | many to many | `res.partner` | computed by rule `_compute_mail_template_fields` (not stored) |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_selection_target_model` | internal rule | self | `mail` | model |  |
| `_selection_languages` | internal rule | self | `mail` | model |  |
| `_compute_no_record` | computation | self | `mail` | depends: `model_id` |  |
| `_compute_mail_template_fields` | computation | self | `mail` | depends: `lang`, `resource_ref` | Preview the mail template (body, subject, ...) depending of the language and the record reference, more precisely the record id for the defined model of the mail template. If no record id is selectable/set, the inline_template placeholders won't be replace in the display information. |
| `_compute_has_attachments` | computation | self | `mail` | depends: `attachment_ids` |  |
| `_compute_has_several_languages_installed` | computation | self | `mail` | depends: `lang` |  |
| `_compute_resource_ref` | computation | self | `mail` | depends: `mail_template_id` |  |
| `_set_mail_attributes` | internal rule | self, values | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `mail` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_template_preview_view_form` | form |  | `error_msg`, `mail_template_id`, `no_record`, `resource_ref`, `has_several_languages_installed`, `lang`, `subject`, `email_from`, `partner_ids`, `email_to`, `email_cc`, `reply_to`, `scheduled_date`, `body_html`, `has_attachments`, `attachment_ids` | `Close` |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_template_preview_action` | Template Preview | form |  | `{'default_mail_template_id':active_id}` | new | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.template.preview.json`; views: `../../../schemas/interfaces/views/mail.template.preview.json`.
