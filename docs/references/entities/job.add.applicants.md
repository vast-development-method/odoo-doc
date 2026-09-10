# Add applicants to a job (`job.add.applicants`)

**Transport name:** `job.add.applicants`  
**Storage name:** `job_add_applicants`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_recruitment`

Description: Add applicants to a job

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `applicant_ids` | Applications | many to many | `hr.applicant` | required |
| `job_ids` | Job Positions | many to many | `hr.job` | required |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_add_applicants_to_job` | internal rule | self | `hr_recruitment` |  |  |
| `action_add_applicants_to_job` | user action | self | `hr_recruitment` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_recruitment_interviewer` | no | no | no | no | `hr_recruitment` |
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.job_add_applicants_view_form` | form |  | `job_ids` | `Create Applications`, `Discard` |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/job.add.applicants.json`; views: `../../../schemas/interfaces/views/job.add.applicants.json`.
