# Recruitment Stages (`hr.recruitment.stage`)

**Transport name:** `hr.recruitment.stage`  
**Storage name:** `hr_recruitment_stage`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_recruitment`

Description: Recruitment Stages

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Stage Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `job_ids` | Job Specific | many to many | `hr.job` | Help: Specific jobs that use this stage. Other jobs will not use this stage. |
| `requirements` | Requirements | multi line text |  |  |
| `template_id` | Email Template | many to one | `mail.template` | Help: If set, a message is posted on the applicant using the template when the applicant is set to the stage. |
| `fold` | Folded in Kanban | boolean |  | Help: This stage is folded in the kanban view when there are no records in that stage to display. |
| `hired_stage` | Hired Stage | boolean |  | Help: If checked, this stage is used to determine the hire date of an applicant |
| `rotting_threshold_days` | Days to rot | integer |  | default ; Help: Day count before applicants in this stage become stale.         Set to 0 to disable.  Changing this parameter will not affect the rotting status/date of resources last updated before this change. |
| `legend_blocked` | Red Kanban Label | single line text |  | required; default computed dynamically (lambda self: _('Blocked')); translatable |
| `legend_waiting` | Orange Kanban Label | single line text |  | required; default computed dynamically (lambda self: _('Waiting')); translatable |
| `legend_done` | Green Kanban Label | single line text |  | required; default computed dynamically (lambda self: _('Ready for Next Stage')); translatable |
| `legend_normal` | Grey Kanban Label | single line text |  | required; default computed dynamically (lambda self: _('In Progress')); translatable |
| `is_warning_visible` | Is Warning Visible | boolean |  | computed by rule `_compute_is_warning_visible` (not stored) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `hr_recruitment` | model |  |
| `_compute_is_warning_visible` | computation | self | `hr_recruitment` | depends: `hired_stage` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment` |
| `group_hr_recruitment_user` | no | yes | no | no | `hr_recruitment` |
| `group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_recruitment_stage_tree` | list |  | `sequence`, `name`, `rotting_threshold_days`, `fold`, `hired_stage` |  |  | `hr_recruitment` |
| `hr_recruitment.view_hr_recruitment_stage_kanban` | kanban |  | `name`, `fold` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_stage_form` | form |  | `name`, `sequence`, `template_id`, `fold`, `hired_stage`, `is_warning_visible`, `job_ids`, `rotting_threshold_days`, `legend_normal`, `legend_blocked`, `legend_waiting`, `legend_done`, `requirements` |  |  | `hr_recruitment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_job_stage_act` | Recruitment / Applicants Stages |  | `[]` | `{}` |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_stage_act` | Stages | list,kanban,form |  |  |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/hr.recruitment.stage.json`; views: `../../../schemas/interfaces/views/hr.recruitment.stage.json`.
