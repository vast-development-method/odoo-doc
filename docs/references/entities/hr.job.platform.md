# Job Platforms (`hr.job.platform`)

**Transport name:** `hr.job.platform`  
**Storage name:** `hr_job_platform`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_recruitment`

Description: Job Platforms

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `email` | Email | single line text |  | required; Help: Applications received from this Email won't be linked to a contact.There will be no email address set on the Applicant either. |
| `regex` | Regex | single line text |  | Help: The regex facilitates to extract information from the subject or body of the received email to autopopulate the Applicant's name field |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_email_uniq` | Constraint | `unique (email)` | The Email must be unique, this one already corresponds to another Job Platform. | `hr_recruitment` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `hr_recruitment` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_recruitment` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_job_platform_form` | form |  | `name`, `regex`, `email` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_job_platform_tree` | list |  | `name`, `email`, `regex` |  |  | `hr_recruitment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.action_hr_job_platforms` | Emails | list,form |  |  |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/hr.job.platform.json`; views: `../../../schemas/interfaces/views/hr.job.platform.json`.
