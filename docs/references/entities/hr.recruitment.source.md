# Source of Applicants (`hr.recruitment.source`)

**Transport name:** `hr.recruitment.source`  
**Storage name:** `hr_recruitment_source`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_recruitment`  
**Extended by packages:** `website_hr_recruitment`

Description: Source of Applicants

## Identity and behavior

- Mixins (classical inheritance): `utm.source.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `email` | Email | single line text |  | read only; related through path `alias_id.display_name` |
| `has_domain` | Has Domain | single line text |  | computed by rule `_compute_has_domain` (not stored) |
| `job_id` | Job | many to one | `hr.job` | indexed; on delete of the target: cascade |
| `alias_id` | Alias identifier | many to one | `mail.alias` | on delete of the target: restrict |
| `medium_id` | Medium | many to one | `utm.medium` | default computed dynamically (lambda self: self.env['utm.medium']._fetch_or_create_utm_medium('website')) |
| `campaign_id` | Campaign | many to one | `utm.campaign` |  |
| `url` | Tracker uniform resource locator | single line text |  | computed by rule `_compute_url` (not stored) |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_has_domain` | computation | self | `hr_recruitment` |  |  |
| `create_alias` | operation | self | `hr_recruitment` |  |  |
| `create_and_get_alias` | operation | self | `hr_recruitment` |  |  |
| `unlink` | lifecycle override | self | `hr_recruitment` |  | Cascade delete aliases to avoid useless / badly configured aliases. |
| `_compute_url` | computation | self | `website_hr_recruitment` | depends: `source_id`, `source_id.name`, `job_id`, `job_id.company_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `base.group_user` | no | yes | no | no | `hr_recruitment` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_recruitment_source_tree` | list |  | `has_domain`, `campaign_id`, `source_id`, `medium_id`, `email` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_source_view_search` | search |  | `source_id`, `job_id` |  |  | `hr_recruitment` |
| `website_hr_recruitment.view_hr_recruitment_tree_url` | xpath | `hr_recruitment.hr_recruitment_source_tree` | `url` |  |  | `website_hr_recruitment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.action_hr_job_sources` | Trackers | list |  | `{'search_default_job_id': [active_id], 'default_job_id': active_id}` |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/hr.recruitment.source.json`; views: `../../../schemas/interfaces/views/hr.recruitment.source.json`.
