# Get Refuse Reason (`applicant.get.refuse.reason`)

**Transport name:** `applicant.get.refuse.reason`  
**Storage name:** `applicant_get_refuse_reason`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_recruitment`

Description: Get Refuse Reason

## Identity and behavior

- Mixins (classical inheritance): `mail.composer.mixin`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `refuse_reason_id` | Refuse Reason | many to one | `hr.applicant.refuse.reason` | required; default computed dynamically (_default_refuse_reason_id) |
| `applicant_ids` | Applicant | many to many | `hr.applicant` |  |
| `send_mail` | Send Email | boolean |  | computed by rule `_compute_send_mail` and stored; precomputed before insertion |
| `template_id` | Email Template | many to one | `mail.template` | computed by rule `_compute_template_id` and stored; restricted by domain `[('model', '=', 'hr.applicant')]`; precomputed before insertion |
| `applicant_without_email` | Applicant(s) not having email | multi line text |  | computed by rule `_compute_applicant_without_email` (not stored) |
| `duplicates` | Refuse Duplicate Applications | boolean |  |  |
| `duplicates_count` | Duplicates Count | integer |  | computed by rule `_compute_duplicate_applicant_ids_domain` (not stored) |
| `duplicate_applicant_ids` | Duplicate Applications | many to many | `hr.applicant` | computed by rule `_compute_duplicate_applicant_ids` and stored; association table `applicant_get_refuse_reason_duplicate_applicants_rel` |
| `duplicate_applicant_ids_domain` | Duplicate Applicant Identifiers Domain | binary |  | computed by rule `_compute_duplicate_applicant_ids_domain` (not stored) |
| `attachment_ids` | Attachments | many to many | `ir.attachment` | computed by rule `_compute_from_template_id` and stored |
| `scheduled_date` | Scheduled Date | single line text |  | computed by rule `_compute_from_template_id` and stored; Help: send emails after that date. This date is considered as being in UTC timezone. |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_refuse_reason_id` | preparation rule | self | `hr_recruitment` |  |  |
| `_compute_send_mail` | computation | self | `hr_recruitment` | depends: `refuse_reason_id`, `applicant_without_email`, `template_id` |  |
| `_compute_applicant_without_email` | computation | self | `hr_recruitment` | depends: `applicant_ids` |  |
| `_compute_duplicate_applicant_ids_domain` | computation | self | `hr_recruitment` | depends: `applicant_ids` |  |
| `_compute_duplicate_applicant_ids` | computation | self | `hr_recruitment` | depends: `duplicates`, `duplicate_applicant_ids_domain` |  |
| `_compute_render_model` | computation | self | `hr_recruitment` | depends: `refuse_reason_id` |  |
| `_compute_template_id` | computation | self | `hr_recruitment` | depends: `refuse_reason_id` |  |
| `_compute_from_template_id` | computation | self | `hr_recruitment` | depends: `template_id` |  |
| `action_refuse_reason_apply` | user action | self | `hr_recruitment` |  |  |
| `_get_related_original_applicants` | preparation rule | self | `hr_recruitment` |  |  |
| `_prepare_send_refusal_mails` | preparation rule | self | `hr_recruitment` |  |  |
| `_prepare_mail_values` | preparation rule | self, applicant | `hr_recruitment` |  | Create mail specific for recipient |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_refuse_reason_apply` | UserError | Unable to post message, please configure the sender's email address. | `hr_recruitment` |
| `action_refuse_reason_apply` | UserError | At least one applicant doesn't have a email; you can't use send email option. | `hr_recruitment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_user` | yes | yes | yes | no | `hr_recruitment` |
| `hr_recruitment.group_hr_recruitment_interviewer` | yes | yes | yes | no | `hr_recruitment` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.applicant_get_refuse_reason_view_form` | form |  | `refuse_reason_id`, `duplicates`, `duplicate_applicant_ids`, `send_mail`, `applicant_ids`, `lang`, `render_model`, `subject`, `can_edit_body`, `body`, `attachment_ids`, `applicant_without_email`, `template_id`, `scheduled_date` | `Refuse`, `Cancel` |  | `hr_recruitment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.applicant_get_refuse_reason_action` | Refuse Reason | form |  |  | new | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/applicant.get.refuse.reason.json`; views: `../../../schemas/interfaces/views/applicant.get.refuse.reason.json`.
