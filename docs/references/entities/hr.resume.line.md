# Resume line of an employee (`hr.resume.line`)

**Transport name:** `hr.resume.line`  
**Storage name:** `hr_resume_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`  
**Extended by packages:** `hr_skills_event`, `hr_skills_slides`, `hr_skills_survey`

Description: Resume line of an employee

## Identity and behavior

- Default ordering: `line_type_id, date_end desc, date_start desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_id` | Employee | many to one | `hr.employee` | required; indexed; on delete of the target: cascade |
| `avatar_128` | Avatar 128 | image |  | related through path `employee_id.avatar_128` |
| `company_id` | Company | many to one |  | related through path `employee_id.company_id` |
| `department_id` | Department | many to one |  | related through path `employee_id.department_id` and stored; extended by packages `hr_skills_survey` |
| `name` | Name | single line text |  | required; translatable |
| `date_start` | Date Start | date |  | required; default computed dynamically (fields.Date.context_today) |
| `date_end` | Date End | date |  |  |
| `duration` | Duration | integer |  | computed by rule `_compute_duration` and stored; extended by packages `hr_skills_slides` |
| `description` | Description | rich text |  | translatable |
| `line_type_id` | Type | many to one | `hr.resume.line.type` |  |
| `is_course` | Is Course | boolean |  | related through path `line_type_id.is_course` |
| `course_type` | Course Type | selection |  | required; default `external`; on delete of the target: {"elearning": "cascade"}; extended by packages `hr_skills_event`, `hr_skills_slides` |
| `color` | Color | single line text |  | computed by rule `_compute_color` (not stored); default `#000000` |
| `external_url` | External uniform resource locator | single line text |  | computed by rule `_compute_external_url` and stored |
| `certificate_filename` | Certificate Filename | single line text |  |  |
| `certificate_file` | Certificate | binary |  |  |
| `resume_line_properties` | Properties | properties |  |  |
| `event_id` | Onsite Course | many to one | `event.event` | read only; computed by rule `_compute_event_id` and stored; indexed (btree_not_null); restricted by domain `[('registration_ids', 'any', [('partner_id.employee', '=', True)])]` |
| `channel_id` | eLearning Course | many to one | `slide.channel` | read only; computed by rule `_compute_channel_id` and stored; indexed (btree_not_null) |
| `course_url` | Course Uniform resource locator | single line text |  | related through path `channel_id.website_absolute_url` |
| `survey_id` | Certification | many to one | `survey.survey` | read only |
| `expiration_status` | Expiration Status | selection |  | computed by rule `_compute_expiration_status` and stored |

## Selection values

### `course_type` (Course Type)

| Value | Label |
|---|---|
| `external` | External |
| `onsite` | Onsite |
| `elearning` | eLearning |

### `expiration_status` (Expiration Status)

| Value | Label |
|---|---|
| `expired` | Expired |
| `expiring` | Expiring |
| `valid` | Valid |

## State fields

State machine fields of this entity: `expiration_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_date_check` | Constraint | `CHECK ((date_start <= date_end OR date_end IS NULL))` | The start date must be anterior to the end date. | `hr_skills` |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_external_url` | on change | self | `hr_skills` | onchange: `external_url` |  |
| `_compute_external_url` | computation | self | `hr_skills` | depends: `course_type` |  |
| `_compute_color` | computation | self | `hr_skills_event`, `hr_skills_slides`, `hr_skills` | depends: `course_type` |  |
| `_onchange_event_id` | on change | self | `hr_skills_event` | onchange: `event_id` |  |
| `_compute_event_id` | computation | self | `hr_skills_event` | depends: `course_type` |  |
| `_compute_duration` | computation | self | `hr_skills_slides` | depends: `channel_id` |  |
| `_onchange_channel_id` | on change | self | `hr_skills_slides` | onchange: `channel_id` |  |
| `_compute_channel_id` | computation | self | `hr_skills_slides` | depends: `course_type` |  |
| `_compute_expiration_status` | computation | self | `hr_skills_survey` | depends: `date_end` |  |
| `copy_data` | lifecycle override | self, default | `hr_skills_survey` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `base.group_user` | yes | yes | yes | yes | `hr_skills` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Resume: employee: read all | `[(4,ref('base.group_user'))]` | `[(1, '=', 1)]` | True | False | False | False |
| Resume: HR user: all | `[(4,ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Resume: employee: create/write/unlink own | `[(4,ref('base.group_user'))]` | `[('employee_id.user_id','=',user.id)]` | False | True | True | True |

## Views (13)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.resume_line_view_form` | form |  | `line_type_id`, `name`, `course_type`, `external_url`, `date_start`, `date_end`, `date_start`, `duration`, `certificate_filename`, `certificate_file`, `resume_line_properties`, `description` |  |  | `hr_skills` |
| `hr_skills.resume_line_view_form_inherit` | xpath | `hr_skills.resume_line_view_form` | `employee_id` |  |  | `hr_skills` |
| `hr_skills.hr_resume_line_list_view` | list |  | `employee_id`, `name`, `date_start`, `course_type`, `line_type_id`, `certificate_filename`, `certificate_file`, `duration`, `external_url`, `description`, `resume_line_properties` |  |  | `hr_skills` |
| `hr_skills.hr_resume_line_kanban_view` | kanban |  | `name`, `course_type`, `employee_id`, `employee_id`, `date_start` |  |  | `hr_skills` |
| `hr_skills.hr_resume_line_calendar_view` | calendar |  | `course_type` |  |  | `hr_skills` |
| `hr_skills.view_resume_lines_filter` | search |  | `company_id`, `department_id` |  |  | `hr_skills` |
| `hr_skills_event.resume_slides_line_view_form` | xpath | `hr_skills.resume_line_view_form` | `event_id` |  |  | `hr_skills_event` |
| `hr_skills_event.resume_slides_line_view_list` | xpath | `hr_skills.hr_resume_line_list_view` |  |  |  | `hr_skills_event` |
| `hr_skills_event.resume_slides_line_view_kanban` | xpath | `hr_skills.hr_resume_line_kanban_view` |  |  |  | `hr_skills_event` |
| `hr_skills_slides.resume_slides_line_view_form` | xpath | `hr_skills.resume_line_view_form` | `channel_id` |  |  | `hr_skills_slides` |
| `hr_skills_slides.resume_slides_line_view_list` | xpath | `hr_skills.hr_resume_line_list_view` |  |  |  | `hr_skills_slides` |
| `hr_skills_slides.resume_slides_line_view_kanban` | xpath | `hr_skills.hr_resume_line_kanban_view` |  |  |  | `hr_skills_slides` |
| `hr_skills_survey.resume_survey_line_view_form` | xpath | `hr_skills.resume_line_view_form` | `survey_id` |  |  | `hr_skills_survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_resume_lines_training_action` | Training Attendances | list,kanban,form,calendar | `[('line_type_id.is_course', '=', True)]` |  |  | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.resume.line.json`; views: `../../../schemas/interfaces/views/hr.resume.line.json`.
