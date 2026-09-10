# Add applicants to talent pool (`talent.pool.add.applicants`)

**Transport name:** `talent.pool.add.applicants`  
**Storage name:** `talent_pool_add_applicants`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_recruitment`

Description: Add applicants to talent pool

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `applicant_ids` | Applicants | many to many | `hr.applicant` | required; restricted by domain `["\|", ["talent_pool_ids", "!=", false], ["is_applicant_in_pool", "=", false]]` |
| `talent_pool_ids` | Talent Pool | many to many | `hr.talent.pool` |  |
| `categ_ids` | Tags | many to many | `hr.applicant.category` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_add_applicants_to_pool` | internal rule | self | `hr_recruitment` |  |  |
| `action_add_applicants_to_pool` | user action | self | `hr_recruitment` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_recruitment_interviewer` | no | no | no | no | `hr_recruitment` |
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.talent_pool_add_applicants_view_form` | form |  | `applicant_ids`, `talent_pool_ids`, `categ_ids` | `Add to Pool`, `Cancel` |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/talent.pool.add.applicants.json`; views: `../../../schemas/interfaces/views/talent.pool.add.applicants.json`.
