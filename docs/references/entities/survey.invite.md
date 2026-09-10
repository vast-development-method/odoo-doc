# Survey Invitation Wizard (`survey.invite`)

**Transport name:** `survey.invite`  
**Storage name:** `survey_invite`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `survey`  
**Extended by packages:** `hr_recruitment_survey`

Description: Survey Invitation Wizard

## Identity and behavior

- Mixins (classical inheritance): `mail.composer.mixin`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `attachment_ids` | Attachments | many to many | `ir.attachment` | computed by rule `_compute_attachment_ids` and stored; association table `survey_mail_compose_message_ir_attachments_rel` |
| `author_id` | Author | many to one | `res.partner` | default computed dynamically (_get_default_author); indexed; on delete of the target: set null |
| `partner_ids` | Recipients | many to many | `res.partner` | restricted by domain `[             '\|', (survey_users_can_signup, '=', 1),             '\|', (not survey_users_login_required, '=', 1),                  ('user_ids', '!=', False),         ]`; association table `survey_invite_partner_ids` |
| `existing_partner_ids` | Existing Partner | many to many | `res.partner` | read only; computed by rule `_compute_existing_partner_ids` (not stored) |
| `emails` | Additional emails | multi line text |  |  |
| `existing_emails` | Existing emails | multi line text |  | read only; computed by rule `_compute_existing_emails` (not stored) |
| `existing_mode` | Handle existing | selection |  | required; default `resend` |
| `existing_text` | Resend Comment | multi line text |  | computed by rule `_compute_existing_text` (not stored) |
| `mail_server_id` | Outgoing mail server | many to one | `ir.mail_server` |  |
| `survey_id` | Survey | many to one | `survey.survey` | required |
| `survey_start_url` | Survey uniform resource locator | single line text |  | computed by rule `_compute_survey_start_url` (not stored) |
| `survey_access_mode` | Survey Access Mode | selection |  | read only; related through path `survey_id.access_mode` |
| `survey_users_login_required` | Survey Users Login Required | boolean |  | read only; related through path `survey_id.users_login_required` |
| `survey_users_can_signup` | Survey Users Can Signup | boolean |  | related through path `survey_id.users_can_signup` |
| `deadline` | Answer deadline | date and time |  |  |
| `send_email` | Send Email | boolean |  | computed by rule `_compute_send_email` (not stored); writable through an inverse rule |
| `applicant_id` | Applicant | many to one | `hr.applicant` |  |

## Selection values

### `existing_mode` (Handle existing)

| Value | Label |
|---|---|
| `new` | New invite |
| `resend` | Resend invite |

## Operations (18)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_author` | preparation rule | self | `survey` | model |  |
| `_compute_send_email` | computation | self | `survey` | depends: `survey_access_mode` |  |
| `_inverse_send_email` | inverse computation | self | `survey` |  |  |
| `_compute_existing_partner_ids` | computation | self | `survey` | depends: `partner_ids`, `survey_id` |  |
| `_compute_existing_emails` | computation | self | `survey` | depends: `emails`, `survey_id` |  |
| `_compute_existing_text` | computation | self | `survey` | depends: `existing_partner_ids`, `existing_emails` |  |
| `_compute_survey_start_url` | computation | self | `survey` | depends: `survey_id.access_token` |  |
| `_compute_render_model` | computation | self | `survey` | depends: `survey_id` |  |
| `_onchange_emails` | on change | self | `survey` | onchange: `emails` |  |
| `_onchange_partner_ids` | on change | self | `survey` | onchange: `partner_ids` |  |
| `create` | lifecycle override | self, vals_list | `survey` | model_create_multi |  |
| `_compute_subject` | computation | self | `survey` | depends: `template_id` |  |
| `_compute_attachment_ids` | computation | self | `survey` | depends: `template_id` | 'OnChange-like' behavior used for template selection: not intended to update records when     individual attachments get added |
| `_prepare_answers` | preparation rule | self, partners, emails | `survey` |  |  |
| `_get_done_partners_emails` | preparation rule | self, existing_answers | `hr_recruitment_survey`, `survey` |  |  |
| `_get_answers_values` | preparation rule | self | `survey` |  |  |
| `_send_mail` | internal rule | self, answer | `hr_recruitment_survey`, `survey` |  | Create mail specific for recipient containing notably its access token |
| `action_invite` | user action | self | `hr_recruitment_survey`, `survey` |  | Process the wizard content and proceed with sending the related email(s), rendering any template patterns on the fly if needed |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_onchange_emails` | UserError | This survey does not allow external people to participate. You should create user accounts or update survey access mode accordingly. | `survey` |
| `_onchange_emails` | UserError | Some emails you just entered are incorrect: %s | `survey` |
| `_onchange_partner_ids` | UserError | The following recipients have no user account: %s. You should create user accounts for them or allow external signup in configuration. | `survey` |
| `_send_mail` | UserError | Unable to post message, please configure the sender's email address. | `survey` |
| `action_invite` | UserError | Please enter at least one valid recipient. | `survey` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_user` | yes | yes | yes | no | `hr_recruitment_survey` |
| `hr_recruitment.group_hr_recruitment_interviewer` | yes | yes | yes | no | `hr_recruitment_survey` |
| `survey.group_survey_user` | yes | yes | yes | no | `survey` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Survey invite: recruitment manager: all recruitment | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 0 |
| Survey invite: recruitment officer: unrestricted or in restricted users | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[                 '&', ('survey_id.survey_type', '=', 'recruitment'),                 '\|',  ('survey_id.restrict_user_ids', 'in', user.id),                         ('survey_id.restrict_user_ids', '=', False)]` | 1 | 1 | 1 | 0 |
| Survey invite: recruitment interviewer: send surveys to applicants for which they are set as interviewer | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[('survey_id.survey_type', '=', 'recruitment'),                 '\|', ('survey_id.hr_job_ids.interviewer_ids', 'in', user.id),                      ('survey_id.hr_job_ids.application_ids.interviewer_ids', 'in', user.id)                 ]` | 1 | 1 | 1 | 0 |
| Survey invite: officer: unrestricted or in restricted users | `[(4, ref('group_survey_user'))]` | `['\|',  ('survey_id.restrict_user_ids', 'in', user.id),                 ('survey_id.restrict_user_ids', '=', False)]` | 1 | 1 | 1 | 0 |
| Survey invite: manager: all | `[(4, ref('group_survey_manager'))]` | `[(1, '=', 1)]` | 1 | 1 | 1 | 0 |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `survey.survey_invite_view_form` | form |  | `author_id`, `survey_access_mode`, `survey_users_login_required`, `survey_users_can_signup`, `survey_id`, `existing_mode`, `lang`, `render_model`, `survey_start_url`, `send_email`, `partner_ids`, `emails`, `existing_partner_ids`, `existing_emails`, `existing_text`, `existing_mode`, `subject`, `can_edit_body`, `body`, `attachment_ids`, `deadline`, `template_id` | `Send`, `Close` |  | `survey` |

Machine-readable definition: `../../../schemas/data/entities/survey.invite.json`; views: `../../../schemas/interfaces/views/survey.invite.json`.
