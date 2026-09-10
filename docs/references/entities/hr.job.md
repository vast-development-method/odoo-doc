# Job Position (`hr.job`)

**Transport name:** `hr.job`  
**Storage name:** `hr_job`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`  
**Extended by packages:** `hr_recruitment`, `hr_skills`, `hr_recruitment_skills`, `hr_recruitment_survey`, `website_hr_recruitment`

Description: Job Position

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.alias.mixin`, `mail.activity.mixin`, `website.seo.metadata`, `website.published.multi.mixin`, `website.searchable.mixin`
- Default ordering: `sequence, name asc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (49)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Job Position | single line text |  | required; translatable; indexed (trigram) |
| `sequence` | Sequence | integer |  | default `10` |
| `expected_employees` | Total Forecasted Employees | integer |  | computed by rule `_compute_employees` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer,hr.group_hr_user`; Help: Expected number of employees for this job position after new recruitment.; extended by packages `hr_recruitment` |
| `no_of_employee` | Current Number of Employees | integer |  | computed by rule `_compute_employees` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer,hr.group_hr_user`; Help: Number of employees currently occupying this job position.; extended by packages `hr_recruitment` |
| `no_of_recruitment` | Target | integer |  | default `1`; not copied on duplication; Help: Number of new employees you expect to recruit. |
| `employee_ids` | Employees | one to many | `hr.employee` | visible only to groups `base.group_user`; inverse field `job_id` |
| `description` | Job Description | rich text |  | translatable; extended by packages `website_hr_recruitment` |
| `requirements` | Requirements | multi line text |  | visible only to groups `hr_recruitment.group_hr_recruitment_interviewer,hr.group_hr_user`; extended by packages `hr_recruitment` |
| `user_id` | Recruiter | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread; visible only to groups `hr_recruitment.group_hr_recruitment_interviewer,hr.group_hr_user`; restricted by domain `[('share', '=', False), ('company_ids', '=?', company_id)]`; Help: The Recruiter will be the default value for all Applicants in this job             position. The Recruiter is automatically added to all meetings with the Applicant.; extended by packages `hr_recruitment` |
| `allowed_user_ids` | Allowed User | many to many | `res.users` | read only; computed by rule `_compute_allowed_user_ids` (not stored) |
| `department_id` | Department | many to one | `hr.department` | changes are tracked in the message thread; indexed (btree_not_null); must belong to the same company |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); changes are tracked in the message thread; restricted by domain `lambda self: [('id', 'in', self.env.companies.ids)]` |
| `contract_type_id` | Employment Type | many to one | `hr.contract.type` | changes are tracked in the message thread |
| `address_id` | Job Location | many to one | `res.partner` | default computed dynamically (_default_address_id); changes are tracked in the message thread; restricted by domain `lambda self: self._address_id_domain()`; Help: Select the location where the applicant will work. Addresses listed here are defined on the company's contact information. |
| `application_ids` | Job Applications | one to many | `hr.applicant` | visible only to groups `hr_recruitment.group_hr_recruitment_interviewer`; inverse field `job_id` |
| `application_count` | Application Count | integer |  | computed by rule `_compute_application_count` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `open_application_count` | Open Application Count | integer |  | computed by rule `_compute_open_application_count` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer`; Help: Number of applications that are still ongoing (not hired or refused) |
| `all_application_count` | All Application Count | integer |  | computed by rule `_compute_all_application_count` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `new_application_count` | New Application | integer |  | computed by rule `_compute_new_application_count` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer`; Help: Number of applications that are new in the flow (typically at first step of the flow) |
| `old_application_count` | Old Application | integer |  | computed by rule `_compute_old_application_count` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `applicant_hired` | Applicants Hired | integer |  | computed by rule `_compute_applicant_hired` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `manager_id` | Department Manager | many to one | `hr.employee` | read only; related through path `department_id.manager_id` and stored; visible only to groups `hr_recruitment.group_hr_recruitment_interviewer,hr.group_hr_user` |
| `document_ids` | Documents | one to many | `ir.attachment` | read only; computed by rule `_compute_document_ids` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `documents_count` | Document Count | integer |  | computed by rule `_compute_document_ids` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `employee_count` | Employee Count | integer |  | computed by rule `_compute_employee_count` (not stored) |
| `alias_id` | Alias | many to one |  | visible only to groups `hr_recruitment.group_hr_recruitment_interviewer`; Help: Email alias for this job position. New emails will automatically create new applicants for this job position. |
| `color` | Color Index | integer |  |  |
| `is_favorite` | Is Favorite | boolean |  | computed by rule `_compute_is_favorite` (not stored); writable through an inverse rule |
| `favorite_user_ids` | Favorite User | many to many | `res.users` | default computed dynamically (_get_default_favorite_user_ids); association table `job_favorite_user_rel` |
| `interviewer_ids` | Interviewers | many to many | `res.users` | visible only to groups `hr_recruitment.group_hr_recruitment_interviewer`; restricted by domain `[('share', '=', False), ('company_ids', '=?', company_id)]`; Help: The Interviewers set on the job position can see all Applicants in it. They have access to the information, the attachments, the meeting management and they can refuse him. You don't need to have Recruitment rights to be set as an interviewer. |
| `extended_interviewer_ids` | Extended Interviewer | many to many | `res.users` | computed by rule `_compute_extended_interviewer_ids` and stored; visible only to groups `hr_recruitment.group_hr_recruitment_interviewer`; association table `hr_job_extended_interviewer_res_users` |
| `industry_id` | Industry | many to one | `res.partner.industry` | changes are tracked in the message thread; visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `expected_degree` | Expected Degree | many to one | `hr.recruitment.degree` | visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `activity_count` | Activity Count | integer |  | computed by rule `_compute_activities` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `job_properties` | Properties | properties |  | visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `applicant_properties_definition` | Applicant Properties | properties definition |  | visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `no_of_hired_employee` | Hired | integer |  | computed by rule `_compute_no_of_hired_employee` and stored; not copied on duplication; visible only to groups `hr_recruitment.group_hr_recruitment_interviewer`; Help: Number of hired employees for this job position during recruitment phase. |
| `job_source_ids` | Job Source | one to many | `hr.recruitment.source` | visible only to groups `hr_recruitment.group_hr_recruitment_interviewer`; inverse field `job_id` |
| `job_skill_ids` | Skills | one to many | `hr.job.skill` | restricted by domain `[["skill_type_id.active", "=", true]]`; inverse field `job_id` |
| `current_job_skill_ids` | Current Job Skill | one to many | `hr.job.skill` | computed by rule `_compute_current_job_skill_ids` (not stored); searchable through a search rule |
| `skill_ids` | Skill | many to many | `hr.skill` | computed by rule `_compute_skill_ids` and stored |
| `applicant_matching_score` | Matching Score(%) | float |  | computed by rule `_compute_applicant_matching_score` (not stored); visible only to groups `hr_recruitment.group_hr_recruitment_interviewer` |
| `survey_id` | Interview Form | many to one | `survey.survey` | indexed (btree_not_null); Help: Choose an interview form for this job position and you will be able to print/answer this interview from all applicants who apply for this job |
| `website_published` | Website Published | boolean |  | changes are tracked in the message thread; Help: Set if the application is published on the website of the company. |
| `website_description` | Website description | rich text |  | default computed dynamically (_get_default_website_description); translatable |
| `job_details` | Process Details | rich text |  | default computed dynamically (_get_default_job_details); translatable; Help: Complementary information that will appear on the job submission page |
| `published_date` | Published Date | date |  | computed by rule `_compute_published_date` and stored |
| `full_url` | job uniform resource locator | single line text |  | computed by rule `_compute_full_url` (not stored) |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_company_uniq` | Constraint | `unique(name, company_id, department_id)` | The name of the job position must be unique per department in company! | `hr` |
| `_no_of_recruitment_positive` | Constraint | `CHECK(no_of_recruitment >= 0)` | The expected number of new employees must be positive. | `hr` |

## Operations (46)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_employees` | computation | self | `hr` | depends: `no_of_recruitment`, `employee_ids.job_id`, `employee_ids.active` |  |
| `_compute_allowed_user_ids` | computation | self | `hr` | depends: `company_id` |  |
| `create` | lifecycle override | self, vals_list | `hr_recruitment`, `hr_skills`, `hr` | model_create_multi | We don't want the current user to be follower of all created job |
| `copy_data` | lifecycle override | self, default | `hr` |  |  |
| `write` | lifecycle override | self, vals | `hr_recruitment`, `hr_skills`, `hr` |  |  |
| `_default_address_id` | preparation rule | self | `hr_recruitment` | model |  |
| `_address_id_domain` | internal rule | self | `hr_recruitment` |  |  |
| `_get_default_favorite_user_ids` | preparation rule | self | `hr_recruitment` |  |  |
| `_compute_no_of_hired_employee` | computation | self | `hr_recruitment` | depends: `application_ids.date_closed` |  |
| `_compute_activities` | computation | self | `hr_recruitment` | depends_context: `uid` |  |
| `_compute_extended_interviewer_ids` | computation | self | `hr_recruitment` | depends: `application_ids.interviewer_ids` |  |
| `_compute_is_favorite` | computation | self | `hr_recruitment` |  |  |
| `_inverse_is_favorite` | inverse computation | self | `hr_recruitment` |  |  |
| `_compute_document_ids` | computation | self | `hr_recruitment` |  |  |
| `_compute_all_application_count` | computation | self | `hr_recruitment` |  |  |
| `_compute_application_count` | computation | self | `hr_recruitment` |  |  |
| `_compute_open_application_count` | computation | self | `hr_recruitment` |  |  |
| `_compute_employee_count` | computation | self | `hr_recruitment` |  |  |
| `_get_first_stage` | preparation rule | self | `hr_recruitment` |  |  |
| `_compute_new_application_count` | computation | self | `hr_recruitment` |  |  |
| `_compute_applicant_hired` | computation | self | `hr_recruitment` |  |  |
| `_compute_old_application_count` | computation | self | `hr_recruitment` | depends: `application_count`, `new_application_count` |  |
| `_alias_get_creation_values` | internal rule | self | `hr_recruitment` |  |  |
| `_order_field_to_sql` | internal rule | self, alias, field_name, direction, nulls, query | `hr_recruitment` |  |  |
| `_creation_subtype` | internal rule | self | `hr_recruitment` |  |  |
| `action_open_attachments` | user action | self | `hr_recruitment` |  |  |
| `action_open_activities` | user action | self | `hr_recruitment` |  |  |
| `_action_load_recruitment_scenario` | internal rule | self | `hr_recruitment` | model |  |
| `action_open_employees` | user action | self | `hr_recruitment` |  |  |
| `_compute_current_job_skill_ids` | computation | self | `hr_skills` | depends: `job_skill_ids` |  |
| `_search_current_job_skill_ids` | search rule | self, operator, value | `hr_skills` |  |  |
| `_compute_skill_ids` | computation | self | `hr_skills` | depends: `job_skill_ids.skill_id` |  |
| `_compute_applicant_matching_score` | computation | self | `hr_recruitment_skills` | depends_context: `active_applicant_id` |  |
| `action_search_matching_applicants` | user action | self | `hr_recruitment_skills` |  |  |
| `action_test_survey` | user action | self | `hr_recruitment_survey` |  |  |
| `action_new_survey` | user action | self | `hr_recruitment_survey` |  |  |
| `_get_default_website_description` | preparation rule | self | `website_hr_recruitment` |  |  |
| `_get_default_job_details` | preparation rule | self | `website_hr_recruitment` |  |  |
| `_compute_full_url` | computation | self | `website_hr_recruitment` | depends: `website_url` |  |
| `_compute_published_date` | computation | self | `website_hr_recruitment` | depends: `website_published` |  |
| `_onchange_website_published` | on change | self | `website_hr_recruitment` | onchange: `website_published` |  |
| `_compute_website_url` | computation | self | `website_hr_recruitment` |  |  |
| `set_open` | operation | self | `website_hr_recruitment` |  |  |
| `get_backend_menu_id` | operation | self | `website_hr_recruitment` |  |  |
| `action_archive` | lifecycle override | self | `website_hr_recruitment` |  |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_hr_recruitment` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | yes | `hr` |
| `base.group_user` | no | yes | no | no | `hr` |
| `group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment` |
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.group_hr_user` | no | yes | no | no | `hr_recruitment` |
| `base.group_public` | no | yes | no | no | `website_hr_recruitment` |
| `base.group_portal` | no | yes | no | no | `website_hr_recruitment` |
| `base.group_user` | no | yes | no | no | `website_hr_recruitment` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Job multi company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| User: All Applicants | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Job Positions: Public | `[(4, ref('base.group_public'))]` | `[('website_published', '=', True)]` | True | False | False | False |
| Job Positions: Portal | `[(4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | True | False | False | False |
| Job Positions: HR Officer | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | True | False | False | False |

## Views (24)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.view_hr_job_form` | form |  | `active`, `name`, `user_id`, `no_of_recruitment`, `company_id`, `department_id`, `contract_type_id`, `description` |  |  | `hr` |
| `hr.view_hr_job_tree` | list |  | `sequence`, `name`, `no_of_employee` |  |  | `hr` |
| `hr.hr_job_view_kanban` | kanban |  | `name`, `department_id`, `expected_employees` |  |  | `hr` |
| `hr.view_job_filter` | search |  | `name`, `department_id` |  | `Unread Messages`, `Archived`, `Department`, `Company`, `Employment Type` | `hr` |
| `hr_recruitment.view_hr_job_kanban` | kanban |  | `active`, `alias_email`, `company_id`, `color`, `is_favorite`, `name`, `alias_id`, `user_id`, `user_id`, `company_id`, `user_id`, `no_of_recruitment`, `new_application_count`, `open_application_count`, `activity_count`, `is_favorite`, `name`, `alias_id`, `user_id`, `company_id`, `user_id`, `user_id`, `application_count` | `action_open_activities`, `%(action_hr_job_applications)d`,  |  | `hr_recruitment` |
| `hr_recruitment.view_job_filter_recruitment` | xpath | `hr.view_job_filter` |  |  | `My Favorites` | `hr_recruitment` |
| `hr_recruitment.hr_job_simple_form` | form |  | `name`, `alias_id`, `alias_name`, `alias_domain_id` | `Create`, `Discard` |  | `hr_recruitment` |
| `hr_recruitment.hr_job_survey` | xpath | `hr.view_hr_job_form` |  |  |  | `hr_recruitment` |
| `hr_recruitment.hr_job_search_view` | xpath | `hr_recruitment.view_job_filter_recruitment` | `company_id`, `department_id` |  | `My Job Positions` | `hr_recruitment` |
| `hr_recruitment.hr_job_view_tree_inherit` | field | `hr.view_hr_job_tree` | `name`, `department_id`, `open_application_count`, `no_of_recruitment` |  |  | `hr_recruitment` |
| `hr_recruitment_skills.hr_job_list_inherit_hr_recruitment_skills` | field | `hr_recruitment.hr_job_view_tree_inherit` | `department_id`, `applicant_matching_score` |  |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.view_hr_job_form` | xpath | `hr_skills.view_hr_job_form` |  |  |  | `hr_recruitment_skills` |
| `hr_recruitment_survey.hr_job_survey_inherit` | field | `hr_recruitment.hr_job_survey` | `interviewer_ids`, `survey_id` |  |  | `hr_recruitment_survey` |
| `hr_recruitment_survey.view_hr_job_kanban_inherit` | xpath | `hr_recruitment.view_hr_job_kanban` | `survey_id` |  |  | `hr_recruitment_survey` |
| `hr_skills.view_hr_job_form` | page | `hr.view_hr_job_form` |  |  |  | `hr_skills` |
| `website_hr_recruitment.view_hr_job_form_website_published_button` | div | `hr_recruitment.hr_job_survey` | `is_published` |  |  | `website_hr_recruitment` |
| `website_hr_recruitment.view_hr_job_form_inherit_website` | field | `hr.view_hr_job_form` | `description` |  |  | `website_hr_recruitment` |
| `website_hr_recruitment.view_hr_job_tree_inherit_website` | field | `hr_recruitment.hr_job_view_tree_inherit` | `no_of_employee`, `website_id` |  |  | `website_hr_recruitment` |
| `website_hr_recruitment.hr_job_website_inherit` | xpath | `hr_recruitment.view_hr_job_kanban` |  |  |  | `website_hr_recruitment` |
| `website_hr_recruitment.hr_job_form_inherit` | xpath | `hr.view_hr_job_form` | `job_details` |  |  | `website_hr_recruitment` |
| `website_hr_recruitment.view_hr_job_kanban_referal_extends` | xpath | `hr_recruitment.view_hr_job_kanban` | `full_url` |  |  | `website_hr_recruitment` |
| `website_hr_recruitment.hr_job_search_view_inherit` | xpath | `hr.view_job_filter` |  |  | `Published` | `website_hr_recruitment` |
| `website_hr_recruitment.job_pages_tree_view` | xpath | `hr.view_hr_job_tree` |  |  |  | `website_hr_recruitment` |
| `website_hr_recruitment.job_pages_kanban_view` | kanban | `hr_job_website_inherit` |  |  |  | `website_hr_recruitment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.action_create_job_position` | Create a Job Position | form |  |  | current | `hr` |
| `hr.action_hr_job` | Job Positions | list,form |  | `{"search_default_Current":1}` |  | `hr` |
| `hr_recruitment.create_job_simple` | Create a Job Position | form |  | `{'dialog_size' : 'medium'}` | new | `hr_recruitment` |
| `hr_recruitment.action_hr_job_config` | Job Positions | list,kanban,form |  | `{'search_default_in_recruitment': 1}` |  | `hr_recruitment` |
| `hr_recruitment.action_hr_job` | Job Positions | kanban,list,form |  | `{}` |  | `hr_recruitment` |
| `hr_recruitment.action_hr_job_interviewer` | Job Positions | kanban,form | `[             '\|',                 ('interviewer_ids', 'in', uid),                 ('extended_interviewer_ids', 'in', uid),         ]` | `{'create': False}` |  | `hr_recruitment` |
| `hr_recruitment_skills.action_find_matching_job` | Matching Positions | list |  | `{'active_applicant_id': active_id,}` | current | `hr_recruitment_skills` |
| `website_hr_recruitment.action_job_pages_list` | Job Pages | list,kanban,form |  | `{'create_action': '/jobs/add'}` |  | `website_hr_recruitment` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `hr_recruitment.action_load_demo_data` | Load demo data | code |  | yes |
| `hr_recruitment_skills.action_applicant_search_applicant` | Search Matching Applicants | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/hr.job.json`; views: `../../../schemas/interfaces/views/hr.job.json`.
