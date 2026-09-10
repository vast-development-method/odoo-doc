# Refuse Reason of Applicant (`hr.applicant.refuse.reason`)

**Transport name:** `hr.applicant.refuse.reason`  
**Storage name:** `hr_applicant_refuse_reason`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_recruitment`

Description: Refuse Reason of Applicant

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10`; not copied on duplication |
| `name` | Description | single line text |  | required; translatable |
| `template_id` | Email Template | many to one | `mail.template` | restricted by domain `[('model', '=', 'hr.applicant')]` |
| `active` | Active | boolean |  | default `True` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment` |
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_applicant_refuse_reason_view_form` | form |  | `name`, `active`, `template_id` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_refuse_reason_view_tree` | list |  | `sequence`, `name`, `template_id` |  |  | `hr_recruitment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_applicant_refuse_reason_action` | Refuse Reasons | list,form |  |  |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/hr.applicant.refuse.reason.json`; views: `../../../schemas/interfaces/views/hr.applicant.refuse.reason.json`.
