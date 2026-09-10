# Send mails to applicants (`applicant.send.mail`)

**Transport name:** `applicant.send.mail`  
**Storage name:** `applicant_send_mail`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_recruitment`

Description: Send mails to applicants

## Identity and behavior

- Mixins (classical inheritance): `mail.composer.mixin`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `applicant_ids` | Applications | many to many | `hr.applicant` | required |
| `author_id` | Author | many to one | `res.partner` | required; default computed dynamically (lambda self: self.env.user.partner_id.id) |
| `attachment_ids` | Attachments | many to many | `ir.attachment` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_render_model` | computation | self | `hr_recruitment` | depends: `subject` |  |
| `action_send` | user action | self | `hr_recruitment` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_user` | yes | yes | yes | no | `hr_recruitment` |
| `hr_recruitment.group_hr_recruitment_interviewer` | yes | yes | yes | no | `hr_recruitment` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.applicant_send_mail_view_form` | form |  | `author_id`, `lang`, `render_model`, `template_id`, `subject`, `applicant_ids`, `body`, `attachment_ids`, `template_id` | `Send`, `Cancel` |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/applicant.send.mail.json`; views: `../../../schemas/interfaces/views/applicant.send.mail.json`.
