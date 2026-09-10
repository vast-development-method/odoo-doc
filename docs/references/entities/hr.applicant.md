# Applicant (`hr.applicant`)

**Transport name:** `hr.applicant`  
**Storage name:** `hr_applicant`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_recruitment`  
**Extended by packages:** `hr_recruitment_skills`, `hr_recruitment_sms`, `hr_recruitment_survey`, `website_hr_recruitment`

Description: Applicant

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.cc`, `mail.thread.main.attachment`, `mail.thread.blacklist`, `mail.thread.phone`, `mail.activity.mixin`, `utm.mixin`, `mail.tracking.duration.mixin`
- Default ordering: `priority desc, sequence, id desc`
- Display name field: `partner_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (69)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10`; indexed |
| `active` | Active | boolean |  | default `True`; indexed; Help: If the active field is set to false, it will allow you to hide the case without removing it. |
| `partner_id` | Contact | many to one | `res.partner` | indexed (btree_not_null); not copied on duplication |
| `partner_name` | Applicant's Name | single line text |  |  |
| `email_from` | Email | single line text |  | computed by rule `_compute_partner_phone_email` and stored; writable through an inverse rule; indexed (trigram); maximum length 128 |
| `email_normalized` | Email Normalized | single line text |  | indexed (trigram) |
| `partner_phone` | Phone | single line text |  | computed by rule `_compute_partner_phone_email` and stored; writable through an inverse rule; indexed (btree_not_null); maximum length 32 |
| `partner_phone_sanitized` | Sanitized Phone Number | single line text |  | computed by rule `_compute_partner_phone_sanitized` and stored; indexed (btree_not_null) |
| `linkedin_profile` | LinkedIn Profile | single line text |  | indexed (btree_not_null) |
| `type_id` | Degree | many to one | `hr.recruitment.degree` |  |
| `availability` | Availability | date |  | changes are tracked in the message thread; Help: The date at which the applicant will be available to start working |
| `color` | Color Index | integer |  | default  |
| `employee_id` | Employee | many to one | `hr.employee` | indexed (btree_not_null); not copied on duplication; Help: Employee linked to the applicant. |
| `emp_is_active` | Employee Active | boolean |  | related through path `employee_id.active` |
| `employee_name` | Employee Name | single line text |  | related through path `employee_id.name` |
| `probability` | Probability | float |  |  |
| `create_date` | Applied on | date and time |  | read only |
| `stage_id` | Stage | many to one | `hr.recruitment.stage` | computed by rule `_compute_stage` and stored; changes are tracked in the message thread; indexed; not copied on duplication; on delete of the target: restrict; restricted by domain `['\|', ('job_ids', '=', False), ('job_ids', '=', job_id)]` |
| `last_stage_id` | Last Stage | many to one | `hr.recruitment.stage` | Help: Stage of the applicant before being in the current stage. Used for lost cases analysis. |
| `categ_ids` | Tags | many to many | `hr.applicant.category` |  |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company` and stored; changes are tracked in the message thread; restricted by domain `lambda self: [('id', 'in', self.env.companies.ids)]` |
| `user_id` | Recruiter | many to one | `res.users` | computed by rule `_compute_user` and stored; changes are tracked in the message thread; restricted by domain `[('share', '=', False), ('company_ids', 'in', company_id)]` |
| `date_closed` | Hire Date | date and time |  | computed by rule `_compute_date_closed` and stored; changes are tracked in the message thread; not copied on duplication |
| `date_open` | Assigned | date and time |  | read only |
| `date_last_stage_update` | Last Stage Update | date and time |  | default computed dynamically (fields.Datetime.now); indexed |
| `priority` | Evaluation | selection |  | default `0` |
| `job_id` | Job Position | many to one | `hr.job` | changes are tracked in the message thread; indexed; not copied on duplication; restricted by domain `company_id and [('company_id', '=', company_id)] or []` |
| `salary_proposed_extra` | Proposed Salary Extra | single line text |  | changes are tracked in the message thread; visible only to groups `hr_recruitment.group_hr_recruitment_user`; Help: Salary Proposed by the Organisation, extra advantages |
| `salary_expected_extra` | Expected Salary Extra | single line text |  | changes are tracked in the message thread; visible only to groups `hr_recruitment.group_hr_recruitment_user`; Help: Salary Expected by Applicant, extra advantages |
| `salary_proposed` | Proposed | float |  | changes are tracked in the message thread; visible only to groups `hr_recruitment.group_hr_recruitment_user`; aggregated with avg; Help: Salary Proposed by the Organisation |
| `salary_expected` | Expected | float |  | changes are tracked in the message thread; visible only to groups `hr_recruitment.group_hr_recruitment_user`; aggregated with avg; Help: Salary Expected by Applicant |
| `department_id` | Department | many to one | `hr.department` | computed by rule `_compute_department` and stored; changes are tracked in the message thread; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `day_open` | Days to Open | float |  | computed by rule `_compute_day` (not stored) |
| `day_close` | Days to Close | float |  | computed by rule `_compute_day` (not stored) |
| `delay_close` | Delay to Close | float |  | read only; computed by rule `_compute_delay` and stored; aggregated with avg; Help: Number of days to close |
| `user_email` | User Email | single line text |  | read only; related through path `user_id.email` |
| `attachment_number` | Number of Attachments | integer |  | computed by rule `_get_attachment_number` (not stored) |
| `attachment_ids` | Attachments | one to many | `ir.attachment` | restricted by domain `[["res_model", "=", "hr.applicant"]]`; inverse field `res_id` |
| `kanban_state` | Kanban State | selection |  | required; default `normal`; not copied on duplication |
| `legend_blocked` | Kanban Blocked | single line text |  | related through path `stage_id.legend_blocked` |
| `legend_done` | Kanban Valid | single line text |  | related through path `stage_id.legend_done` |
| `legend_waiting` | Kanban Waiting | single line text |  | related through path `stage_id.legend_waiting` |
| `legend_normal` | Kanban Ongoing | single line text |  | related through path `stage_id.legend_normal` |
| `refuse_reason_id` | Refuse Reason | many to one | `hr.applicant.refuse.reason` | changes are tracked in the message thread |
| `meeting_ids` | Meetings | one to many | `calendar.event` | inverse field `applicant_id` |
| `meeting_display_text` | Meeting Display Text | single line text |  | computed by rule `_compute_meeting_display` (not stored) |
| `meeting_display_date` | Meeting Display Date | date |  | computed by rule `_compute_meeting_display` (not stored) |
| `campaign_id` | Campaign | many to one |  | on delete of the target: set null |
| `medium_id` | Medium | many to one |  | on delete of the target: set null; Help: This displays how the applicant has reached out, e.g. via Email, LinkedIn, Website, etc. |
| `source_id` | Source | many to one |  | on delete of the target: set null |
| `interviewer_ids` | Interviewers | many to many | `res.users` | changes are tracked in the message thread; indexed; not copied on duplication; restricted by domain `[('share', '=', False), ('company_ids', 'in', company_id)]`; association table `hr_applicant_res_users_interviewers_rel` |
| `application_status` | Application Status | selection |  | computed by rule `_compute_application_status` (not stored); searchable through a search rule |
| `application_count` | Application Count | integer |  | computed by rule `_compute_application_count` (not stored); Help: Applications with the same email or phone or mobile |
| `applicant_properties` | Properties | properties |  |  |
| `applicant_notes` | Applicant Notes | rich text |  |  |
| `refuse_date` | Refuse Date | date and time |  |  |
| `talent_pool_ids` | Talent Pools | many to many | `hr.talent.pool` |  |
| `pool_applicant_id` | Pool Applicant | many to one | `hr.applicant` | indexed (btree_not_null) |
| `is_pool_applicant` | Is Pool Applicant | boolean |  | computed by rule `_compute_is_pool` (not stored) |
| `is_applicant_in_pool` | Is Applicant In Pool | boolean |  | computed by rule `_compute_is_applicant_in_pool` (not stored); searchable through a search rule |
| `talent_pool_count` | Talent Pool Count | integer |  | computed by rule `_compute_talent_pool_count` (not stored) |
| `applicant_skill_ids` | Skills | one to many | `hr.applicant.skill` | inverse field `applicant_id` |
| `current_applicant_skill_ids` | Current Applicant Skill | one to many | `hr.applicant.skill` | computed by rule `_compute_current_applicant_skill_ids` (not stored); inverse field `applicant_id` |
| `skill_ids` | Skill | many to many | `hr.skill` | computed by rule `_compute_skill_ids` and stored |
| `matching_skill_ids` | Matching Skills | many to many | `hr.skill` | computed by rule `_compute_matching_skill_ids` (not stored) |
| `missing_skill_ids` | Missing Skills | many to many | `hr.skill` | computed by rule `_compute_matching_skill_ids` (not stored) |
| `matching_score` | Matching Score | integer |  | computed by rule `_compute_matching_skill_ids` (not stored) |
| `survey_id` | Survey | many to one | `survey.survey` | read only; related through path `job_id.survey_id` |
| `response_ids` | Responses | one to many | `survey.user_input` | inverse field `applicant_id` |

## Selection values

### `kanban_state` (Kanban State)

| Value | Label |
|---|---|
| `normal` | In Progress |
| `done` | Ready for Next Stage |
| `waiting` | Waiting |
| `blocked` | Blocked |

### `application_status` (Application Status)

| Value | Label |
|---|---|
| `ongoing` | Ongoing |
| `hired` | Hired |
| `refused` | Refused |
| `archived` | Archived |

## State fields

State machine fields of this entity: `kanban_state`, `application_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_job_id_stage_id_idx` | Index | `(job_id, stage_id) WHERE active IS TRUE` |  | `hr_recruitment` |

## Operations (65)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_talent_pool_required` | validation | self | `hr_recruitment` | constrains: `talent_pool_ids`, `pool_applicant_id` |  |
| `_compute_talent_pool_count` | computation | self | `hr_recruitment` | depends: `email_normalized`, `partner_phone_sanitized`, `linkedin_profile`, `pool_applicant_id.talent_pool_ids` | This method will find the amount of talent pools the current application is associated with. An application can either be associated directly with a talent pool through talent_pool_ids and/or pool_applicant_id.talent_pool_ids or indirectly by having the same email, phone number or linkedin as a directly linked application. |
| `_compute_partner_phone_sanitized` | computation | self | `hr_recruitment` | depends: `partner_phone` |  |
| `_compute_partner_phone_email` | computation | self | `hr_recruitment` | depends: `partner_id` |  |
| `_inverse_partner_email` | inverse computation | self | `hr_recruitment` |  |  |
| `_compute_application_count` | computation | self | `hr_recruitment` | depends: `email_normalized`, `partner_phone_sanitized`, `linkedin_profile` | This method will calculate the number of applications that are either directly or indirectly linked to the current application(s) - An application is considered directly linked if it shares the same   pool_applicant_id - An application is considered indirectly_linked if it has the same   value as the current application(s) in any of the following field:   email, phone number or linkedin  Note: If self has pool_applicant_id, email, phone number or linkedin set this method will include self in the returned count |
| `_compute_is_pool` | computation | self | `hr_recruitment` | depends: `talent_pool_ids` |  |
| `_get_similar_applicants_domain` | preparation rule | self, ignore_talent, only_talent | `hr_recruitment` |  | This method returns a domain for the applicants whitch match with the current applicant according to email_from, partner_phone or linkedin_profile. Thus, search on the domain will return the current applicant as well if any of the following fields are filled.  Args:     ignore_talent: if you want the domain to only include applicants not belonging to a talent pool     only_talent: if you want the domain to only include applicants belonging to a talent pool  Returns:     Domain() |
| `_compute_is_applicant_in_pool` | computation | self | `hr_recruitment` | depends: `talent_pool_ids`, `pool_applicant_id`, `email_normalized`, `partner_phone_sanitized`, `linkedin_profile` | Computes if an application is linked to a talent pool or not. An application can either be directly or indirectly linked to a talent pool. Direct link:     - 1. Application has talent_pool_ids set, meaning this application         is a talent pool application, or talent for short.     - 2. Application has pool_applicant_id set, meaning this application     is a copy or directly linked to a talent (scenario 1)  Indirect link:     - 3. Application shares a phone number, email, or linkedin with a         direclty linked application.  Note: While possible, linking an application to a pool through  |
| `_search_is_applicant_in_pool` | search rule | self, operator, value | `hr_recruitment` |  | This function is needed to hide duplicates when adding applicants/talents to a talent pool. All applications that have either talent_pool_ids or pool_applicant_id set are considered directly in a pool. Furthermore, any application with the same phone number, email or linkedin as the first applications, that are directly in the pool, are also considered to belong to the same talent pool.  Returns:     returns a domain with ids of applications that are either directly or indirectly linked to a pool |
| `_compute_day` | computation | self | `hr_recruitment` | depends: `date_open`, `date_closed` |  |
| `_compute_delay` | computation | self | `hr_recruitment` | depends: `day_open`, `day_close` |  |
| `_get_rotting_depends_fields` | preparation rule | self | `hr_recruitment` |  |  |
| `_get_rotting_domain` | preparation rule | self | `hr_recruitment` |  |  |
| `_compute_meeting_display` | computation | self | `hr_recruitment` | depends_context: `lang`; depends: `meeting_ids`, `meeting_ids.start` |  |
| `_compute_application_status` | computation | self | `hr_recruitment` | depends: `refuse_reason_id`, `date_closed` |  |
| `_search_application_status` | search rule | self, operator, value | `hr_recruitment` |  |  |
| `_get_attachment_number` | preparation rule | self | `hr_recruitment` |  |  |
| `_read_group_stage_ids` | internal rule | self, stages, domain | `hr_recruitment` | model |  |
| `_compute_company` | computation | self | `hr_recruitment` | depends: `job_id`, `department_id`, `job_id.company_id` |  |
| `_compute_department` | computation | self | `hr_recruitment` | depends: `job_id`, `job_id.department_id` |  |
| `_compute_stage` | computation | self | `hr_recruitment` | depends: `job_id` |  |
| `_compute_user` | computation | self | `hr_recruitment` | depends: `job_id` |  |
| `_phone_get_number_fields` | internal rule | self | `hr_recruitment` |  | This method returns the fields to use to find the number to use to send an SMS on a record. |
| `_compute_date_closed` | computation | self | `hr_recruitment` | depends: `stage_id.hired_stage` |  |
| `copy_data` | lifecycle override | self, default | `hr_recruitment` |  |  |
| `create` | lifecycle override | self, vals_list | `hr_recruitment_skills`, `hr_recruitment` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_recruitment_skills`, `hr_recruitment` |  |  |
| `copy` | lifecycle override | self, default | `hr_recruitment` |  |  |
| `get_empty_list_help` | operation | self, help_message | `hr_recruitment` | model |  |
| `get_view` | lifecycle override | self, view_id, view_type, **options | `hr_recruitment` | model |  |
| `action_create_meeting` | user action | self | `hr_recruitment` |  | This opens Meeting's calendar view to schedule meeting on current applicant @return: Dictionary value for created Meeting view |
| `action_open_attachments` | user action | self | `hr_recruitment` |  |  |
| `action_open_employee` | user action | self | `hr_recruitment` |  |  |
| `action_open_applications` | user action | self | `hr_recruitment` |  |  |
| `action_talent_pool_stat_button` | user action | self | `hr_recruitment` |  |  |
| `link_applicant_to_talent` | operation | self | `hr_recruitment` |  |  |
| `action_talent_pool_add_applicants` | user action | self | `hr_recruitment` |  |  |
| `action_job_add_applicants` | user action | self | `hr_recruitment` |  |  |
| `_track_template` | messaging hook | self, changes | `hr_recruitment` |  |  |
| `_creation_subtype` | internal rule | self | `hr_recruitment` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `hr_recruitment` |  |  |
| `_notify_get_reply_to` | internal rule | self, default, author_id | `hr_recruitment` |  | Override to set alias of applicants to their job definition if any. |
| `_get_customer_information` | preparation rule | self | `hr_recruitment` |  |  |
| `_compute_display_name` | computation | self | `hr_recruitment` | depends: `partner_name`; depends_context: `show_partner_name` |  |
| `message_new` | messaging hook | self, msg_dict, custom_values | `hr_recruitment` | model |  |
| `_message_post_after_hook` | messaging hook | self, message, msg_vals | `hr_recruitment` |  |  |
| `create_employee_from_applicant` | operation | self | `hr_recruitment` |  | Create an employee from applicant |
| `_get_employee_create_vals` | preparation rule | self | `hr_recruitment_skills`, `hr_recruitment` |  |  |
| `_check_interviewer_access` | validation | self | `hr_recruitment` |  |  |
| `archive_applicant` | operation | self | `hr_recruitment` |  |  |
| `reset_applicant` | operation | self | `hr_recruitment` |  | Reinsert the applicant into the recruitment pipe in the first stage |
| `action_archive` | lifecycle override | self | `hr_recruitment` |  |  |
| `action_unarchive` | lifecycle override | self | `hr_recruitment` |  |  |
| `action_send_email` | user action | self | `hr_recruitment` |  |  |
| `_get_duration_from_tracking` | preparation rule | self, trackings | `hr_recruitment` |  |  |
| `_compute_current_applicant_skill_ids` | computation | self | `hr_recruitment_skills` | depends: `applicant_skill_ids` |  |
| `_compute_skill_ids` | computation | self | `hr_recruitment_skills` | depends: `applicant_skill_ids.skill_id` |  |
| `_compute_matching_skill_ids` | computation | self | `hr_recruitment_skills` | depends_context: `matching_job_id`; depends: `current_applicant_skill_ids`, `type_id`, `job_id`, `job_id.job_skill_ids`, `job_id.expected_degree` |  |
| `_map_applicant_skill_ids_to_talent_skill_ids` | internal rule | self, vals | `hr_recruitment_skills` |  | The applicant_skills_ids contains a list of ORM tuples i.e (command, record ID, {values}) The challenge lies in the uniqueness of the record ID in this tuple. Each skill (e.g., 'arabic') has a distinct ID per applicant, i.e arabic in applicant 1 will have a different id from arabic in applicant 2. This means the content of applicant_skills_ids is unique for each record and attempting to pass it directly (e.g., applicant.pool_applicant_id.write(vals)) won't yield results so we must update each tuple to have the correct command and record ID for the talent pool applicant  :param vals: list of CR |
| `action_add_to_job` | user action | self | `hr_recruitment_skills` |  |  |
| `action_send_sms` | user action | self | `hr_recruitment_sms` |  |  |
| `action_print_survey` | user action | self | `hr_recruitment_survey` |  | If response is available then print this response otherwise print survey form (print template of the survey) |
| `action_send_survey` | user action | self | `hr_recruitment_survey` |  |  |
| `website_form_input_filter` | operation | self, request, values | `website_hr_recruitment` |  |  |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_talent_pool_required` | ValidationError | Talent must belong to at least one Talent Pool. | `hr_recruitment` |
| `_inverse_partner_email` | UserError | You must define a Contact Name for this applicant. | `hr_recruitment` |
| `copy` | UserError | You cannot duplicate the talent(s). | `hr_recruitment` |
| `action_create_meeting` | UserError | You must define a Contact Name for this applicant. | `hr_recruitment` |
| `create_employee_from_applicant` | UserError | Please provide an applicant name. | `hr_recruitment` |
| `_check_interviewer_access` | UserError | You are not allowed to perform this action. | `hr_recruitment` |
| `action_send_survey` | UserError | Please provide an applicant name. | `hr_recruitment_survey` |
| `website_form_input_filter` | UserError | The job offer has been closed. | `website_hr_recruitment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_recruitment_interviewer` | no | yes | yes | no | `hr_recruitment` |
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Applicant multi company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Applicant Interviewer | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[             '\|',                 ('job_id.interviewer_ids', 'in', user.id),                 ('interviewer_ids', 'in', user.id),         ]` | True | True | False | False |
| User: All Applicants | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (23)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.crm_case_tree_view_job` | list |  | `message_needaction`, `last_stage_id`, `date_last_stage_update`, `partner_name`, `create_date`, `email_from`, `partner_phone`, `job_id`, `stage_id`, `priority`, `categ_ids`, `user_id`, `is_rotting`, `rotting_days`, `type_id`, `application_status`, `refuse_reason_id`, `activity_ids`, `activity_date_deadline`, `interviewer_ids`, `medium_id`, `source_id`, `salary_expected`, `salary_proposed`, `availability`, `department_id`, `company_id`, `company_id` | `Create Applications`, `Add Applicants`, `Refuse` |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_tree_activity` | list |  | `partner_id`, `activity_date_deadline`, `activity_type_id`, `activity_summary`, `stage_id`, `activity_exception_decoration` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_form` | form |  | `company_id`, `application_status`, `employee_id`, `meeting_ids`, `refuse_reason_id`, `email_normalized`, `partner_phone_sanitized`, `emp_is_active`, `is_rotting`, `rotting_days`, `stage_id`, `employee_name`, `application_count`, `meeting_display_text`, `meeting_display_date`, `talent_pool_count`, `active`, `legend_normal`, `legend_blocked`, `legend_waiting`, `legend_done`, `kanban_state`, `partner_name`, `partner_name`, `partner_id`, `refuse_reason_id`, `email_from`, `partner_phone`, `linkedin_profile`, `priority`, `job_id`, `talent_pool_ids`, `user_id`, `interviewer_ids`, `date_closed`, `categ_ids`, `applicant_properties`, `applicant_notes`, `type_id`, `availability`, `department_id`, `company_id`, `salary_expected`, `salary_expected_extra`, `salary_proposed`, `salary_proposed_extra`, `source_id`, `medium_id`, `campaign_id` | `Create Employee`, `Refuse`, `Restore`, `Create Applications`, `Add to Pool`, `action_open_employee`, `action_open_applications`, `action_create_meeting`, `action_talent_pool_stat_button` |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_form_interviewer` | xpath | `hr_applicant_view_form` |  |  |  | `hr_recruitment` |
| `hr_recruitment.crm_case_pivot_view_job` | pivot |  | `job_id`, `stage_id`, `color` |  |  | `hr_recruitment` |
| `hr_recruitment.crm_case_graph_view_job` | graph |  | `stage_id` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_search_bis` | search |  | `partner_name`, `email_from`, `job_id`, `department_id`, `company_id`, `user_id`, `stage_id`, `categ_ids`, `refuse_reason_id`, `application_status`, `date_closed`, `activity_user_id`, `activity_type_id`, `attachment_ids` |  | `My Applications`, `Unassigned`, `Applicants`, `Talents`, `In Progress`, `Hired`, `Ready for Next Stage`, `Waiting`, `Blocked`, `Rotting`, `Directly Available`, `Creation Date`, `Last Stage Update`, `Unread Messages`, `Archived`, `Refused`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Running Applicants`, `Job`, `Stage`, `Responsible`, `Talent Pool`, `Creation Date`, `Hiring Date`, `Last Stage Update`, `Refuse Reason`, `Company`, `Properties` | `hr_recruitment` |
| `hr_recruitment.hr_applicant_calendar_view` | calendar |  | `partner_name`, `job_id`, `priority`, `user_id`, `activity_summary` |  |  | `hr_recruitment` |
| `hr_recruitment.quick_create_applicant_form` | form |  | `partner_name`, `job_id`, `company_id` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_kanban_view_applicant` | kanban |  | `stage_id`, `legend_normal`, `legend_blocked`, `legend_waiting`, `legend_done`, `date_closed`, `color`, `user_id`, `active`, `application_status`, `company_id`, `partner_name`, `job_id`, `categ_ids`, `applicant_properties`, `priority`, `activity_ids`, `is_rotting`, `rotting_days`, `attachment_number`, `kanban_state`, `user_id` | `Add Applicants` |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_activity` | activity |  | `user_id`, `partner_name` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_kanban_view_applicant_talent_pool` | xpath | `hr_recruitment.hr_kanban_view_applicant` |  |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_pivot` | pivot |  | `stage_id`, `job_id`, `partner_name` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_graph` | graph |  | `stage_id`, `job_id` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_search` | search |  | `job_id`, `department_id`, `user_id`, `priority`, `stage_id`, `company_id`, `create_date`, `date_closed` |  | `Creation Date`, `Unassigned`, `New`, `Ongoing`, `Refused`, `Archived`, `Responsible`, `Company`, `Jobs`, `Department`, `Tags`, `Stage`, `Creation Date` | `hr_recruitment` |
| `hr_recruitment_skills.hr_applicant_view_form` | notebook | `hr_recruitment.hr_applicant_view_form` | `id`, `current_applicant_skill_ids`, `skill_type_id`, `skill_id`, `skill_level_id`, `level_progress`, `valid_to`, `matching_score` |  |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.hr_applicant_view_search_bis` | xpath | `hr_recruitment.hr_applicant_view_search_bis` | `applicant_skill_ids` |  |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.hr_applicant_view_search` | xpath | `hr_recruitment.hr_applicant_view_search` | `applicant_skill_ids` |  |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.crm_case_tree_view_job` | field | `hr_recruitment.crm_case_tree_view_job` | `stage_id`, `matching_score` |  |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.crm_case_tree_view_inherit_hr_recruitment_skills` | xpath | `hr_recruitment_skills.crm_case_tree_view_job` |  |  |  | `hr_recruitment_skills` |
| `hr_recruitment_survey.crm_case_tree_view_job_inherit` | xpath | `hr_recruitment.crm_case_tree_view_job` | `survey_id`, `response_ids` |  |  | `hr_recruitment_survey` |
| `hr_recruitment_survey.hr_applicant_view_form_inherit` | xpath | `hr_recruitment.hr_applicant_view_form` |  | `Send Interview` |  | `hr_recruitment_survey` |
| `hr_recruitment_survey.hr_kanban_view_applicant_inherit` | xpath | `hr_recruitment.hr_kanban_view_applicant` | `survey_id` |  |  | `hr_recruitment_survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.action_hr_job_applications` | Applications | kanban,list,form,graph,calendar,pivot,activity |  | `{             'search_default_job_id': [active_id],             'search_default_applicants': 1,             'default_job_id': active_id,             'dialog_size':'medium',             'allow_search_matching_applicants': 1         }` |  | `hr_recruitment` |
| `hr_recruitment.action_hr_talent_pool_applications` | Talents | list,kanban,form,graph,calendar,pivot,activity | `[             ('talent_pool_ids', '=', active_ids),         ]` | `{'default_talent_pool_ids': active_ids}` |  | `hr_recruitment` |
| `hr_recruitment.action_hr_applicant_new` |  | form |  | `{'default_job_id': active_id}` |  | `hr_recruitment` |
| `hr_recruitment.crm_case_categ0_act_job` | Applications | kanban,list,form,pivot,graph,calendar,activity |  | `{'search_default_applicants': 1}` |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_action_from_department` | New Applications | list,kanban,form,graph,calendar,pivot | `[('stage_id.sequence','<=','1')]` | `{             'search_default_department_id': active_id,             'default_department_id': active_id,             'invisible_department': False}` |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_action_analysis` | Recruitment Analysis | graph,pivot |  | `{'search_default_creation_month': 1, 'search_default_job': 2}` |  | `hr_recruitment` |
| `hr_recruitment.action_hr_recruitment_report_filtered_department` | Recruitment Analysis | graph,pivot |  | `{             'search_default_department_id': [active_id],             'default_department_id': active_id}` |  | `hr_recruitment` |
| `hr_recruitment.action_hr_recruitment_report_filtered_job` | Recruitment Analysis | graph,pivot |  | `{             'search_default_creation_month': 1,             'search_default_job_id': [active_id],             'default_job_id': active_id}` |  | `hr_recruitment` |
| `hr_recruitment.action_hr_job_new_application` | New Application | form |  | `{'search_default_job_id': [active_id], 'default_job_id': active_id}` |  | `hr_recruitment` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `hr_recruitment.action_applicant_send_mail` | Send Email | code |  | yes |
| `hr_recruitment_sms.action_applicant_send_sms` | Send SMS | code |  | yes |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `hr_recruitment.email_template_data_applicant_refuse` | Recruitment: Refuse | Your Job Application: {{ object.job_id.name }} |
| `hr_recruitment.email_template_data_applicant_interest` | Recruitment: Interest | Your Job Application: {{ object.job_id.name }} |
| `hr_recruitment.email_template_data_applicant_congratulations` | Recruitment: Application Acknowledgement | Your Job Application: {{ object.job_id.name }} |
| `hr_recruitment.email_template_data_applicant_not_interested` | Recruitment: Not interested anymore | Your Job Application: {{ object.job_id.name }} |

Machine-readable definition: `../../../schemas/data/entities/hr.applicant.json`; views: `../../../schemas/interfaces/views/hr.applicant.json`.
