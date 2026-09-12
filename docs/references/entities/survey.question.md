# Survey Question (`survey.question`)

**Transport name:** `survey.question`  
**Storage name:** `survey_question`  
**Kind:** persistent entity (one table)  
**Defined by package:** `survey`  
**Extended by packages:** `survey_crm`

Description: Survey Question

## Identity and behavior

- Default ordering: `sequence,id`
- Display name field: `title`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (59)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `title` | Title | single line text |  | required; translatable |
| `description` | Description | rich text |  | translatable; Help: Use this field to add additional explanations about your question or to illustrate it with pictures or a video |
| `question_placeholder` | Placeholder | single line text |  | computed by rule `_compute_question_placeholder` and stored; translatable |
| `background_image` | Background Image | image |  | computed by rule `_compute_background_image` and stored |
| `background_image_url` | Background Url | single line text |  | computed by rule `_compute_background_image_url` (not stored) |
| `survey_id` | Survey | many to one | `survey.survey` | indexed (btree_not_null); on delete of the target: cascade |
| `scoring_type` | Scoring Type | selection |  | read only; related through path `survey_id.scoring_type` |
| `sequence` | Sequence | integer |  | default `10` |
| `session_available` | Live Session available | boolean |  | read only; related through path `survey_id.session_available` |
| `survey_session_speed_rating` | Survey Session Speed Rating | boolean |  | related through path `survey_id.session_speed_rating` |
| `survey_session_speed_rating_time_limit` | General Time limit (seconds) | integer |  | related through path `survey_id.session_speed_rating_time_limit` |
| `is_page` | Is a page? | boolean |  |  |
| `question_ids` | Questions | one to many | `survey.question` | computed by rule `_compute_question_ids` (not stored) |
| `questions_selection` | Questions Selection | selection |  | read only; related through path `survey_id.questions_selection`; Help: If randomized is selected, add the number of random questions next to the section. |
| `random_questions_count` | # Questions Randomly Picked | integer |  | default `1`; Help: Used on randomized sections to take X random questions from all the questions of that section. |
| `page_id` | Page | many to one | `survey.question` | computed by rule `_compute_page_id` and stored |
| `question_type` | Question Type | selection |  | computed by rule `_compute_question_type` and stored |
| `is_scored_question` | Scored | boolean |  | computed by rule `_compute_is_scored_question` and stored; Help: Include this question as part of quiz scoring. Requires an answer and answer score to be taken into account. |
| `has_image_only_suggested_answer` | Has image only suggested answer | boolean |  | computed by rule `_compute_has_image_only_suggested_answer` (not stored) |
| `answer_numerical_box` | Correct numerical answer | float |  | Help: Correct number answer for this question. |
| `answer_date` | Correct date answer | date |  | Help: Correct date answer for this question. |
| `answer_datetime` | Correct datetime answer | date and time |  | Help: Correct date and time answer for this question. |
| `answer_score` | Score | float |  | Help: Score value for a correct answer to this question. |
| `save_as_email` | Save as user email | boolean |  | computed by rule `_compute_save_as_email` and stored; Help: If checked, this option will save the user's answer as its email address. |
| `save_as_nickname` | Save as user nickname | boolean |  | computed by rule `_compute_save_as_nickname` and stored; Help: If checked, this option will save the user's answer as its nickname. |
| `suggested_answer_ids` | Types of answers | one to many | `survey.question.answer` | inverse field `question_id`; Help: Labels used for proposed choices: simple choice, multiple choice and columns of matrix |
| `matrix_subtype` | Matrix Type | selection |  | default `simple` |
| `matrix_row_ids` | Matrix Rows | one to many | `survey.question.answer` | inverse field `matrix_question_id`; Help: Labels used for proposed choices: rows of matrix |
| `scale_min` | Scale Minimum Value | integer |  | default  |
| `scale_max` | Scale Maximum Value | integer |  | default `10` |
| `scale_min_label` | Scale Minimum Label | single line text |  | translatable |
| `scale_mid_label` | Scale Middle Label | single line text |  | translatable |
| `scale_max_label` | Scale Maximum Label | single line text |  | translatable |
| `is_time_limited` | The question is limited in time | boolean |  | Help: Currently only supported for live sessions. |
| `is_time_customized` | Customized speed rewards | boolean |  |  |
| `time_limit` | Time limit (seconds) | integer |  |  |
| `comments_allowed` | Show Comments Field | boolean |  |  |
| `comments_message` | Comment Message | single line text |  | translatable |
| `comment_count_as_answer` | Comment is an answer | boolean |  |  |
| `validation_required` | Validate entry | boolean |  | computed by rule `_compute_validation_required` and stored |
| `validation_email` | Input must be an email | boolean |  |  |
| `validation_length_min` | Minimum Text Length | integer |  | default  |
| `validation_length_max` | Maximum Text Length | integer |  | default  |
| `validation_min_float_value` | Minimum value | float |  | default  |
| `validation_max_float_value` | Maximum value | float |  | default  |
| `validation_min_date` | Minimum Date | date |  |  |
| `validation_max_date` | Maximum Date | date |  |  |
| `validation_min_datetime` | Minimum Datetime | date and time |  |  |
| `validation_max_datetime` | Maximum Datetime | date and time |  |  |
| `validation_error_msg` | Validation Error | single line text |  | translatable |
| `constr_mandatory` | Mandatory Answer | boolean |  |  |
| `constr_error_msg` | Error message | single line text |  | translatable |
| `user_input_line_ids` | Answers | one to many | `survey.user_input.line` | visible only to groups `survey.group_survey_user`; restricted by domain `[["skipped", "=", false]]`; inverse field `question_id` |
| `triggering_question_ids` | Triggering Questions | many to many | `survey.question` | computed by rule `_compute_triggering_question_ids` (not stored); Help: Questions containing the triggering answer(s) to display the current question. |
| `allowed_triggering_question_ids` | Allowed Triggering Questions | many to many | `survey.question` | computed by rule `_compute_allowed_triggering_question_ids` (not stored); not copied on duplication |
| `is_placed_before_trigger` | Is misplaced? | boolean |  | computed by rule `_compute_allowed_triggering_question_ids` (not stored); Help: Is this question placed before any of its trigger questions? |
| `triggering_answer_ids` | Triggering Answers | many to many | `survey.question.answer` | not copied on duplication; restricted by domain `[             ('question_id.survey_id', '=', survey_id),             '&', ('question_id.question_type', 'in', ['simple_choice', 'multiple_choice']),                  '\|',                      ('question_id.sequence', '<', sequence),                      '&', ('question_id.sequence', '=', sequence), ('question_id.id', '<', id)         ]`; Help: Picking any of these answers will trigger this question. Leave the field empty if the question should always be displayed. |
| `survey_type` | Survey Type | selection |  | related through path `survey_id.survey_type` |
| `generate_lead` | Lead Generating | boolean |  | computed by rule `_compute_generate_lead` (not stored); Help: At least one of the question answers can generate leads. |

## Selection values

### `question_type` (Question Type)

| Value | Label |
|---|---|
| `simple_choice` | Multiple choice: only one answer |
| `multiple_choice` | Multiple choice: multiple answers allowed |
| `text_box` | Multiple Lines Text Box |
| `char_box` | Single Line Text Box |
| `numerical_box` | Numerical Value |
| `scale` | Scale |
| `date` | Date |
| `datetime` | Datetime |
| `matrix` | Matrix |

### `matrix_subtype` (Matrix Type)

| Value | Label |
|---|---|
| `simple` | One choice per row |
| `multiple` | Multiple choices per row |

## Database constraints and indexes (11)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_positive_len_min` | Constraint | `CHECK (validation_length_min >= 0)` | A length must be positive! | `survey` |
| `_positive_len_max` | Constraint | `CHECK (validation_length_max >= 0)` | A length must be positive! | `survey` |
| `_validation_length` | Constraint | `CHECK (validation_length_min <= validation_length_max)` | Max length cannot be smaller than min length! | `survey` |
| `_validation_float` | Constraint | `CHECK (validation_min_float_value <= validation_max_float_value)` | Max value cannot be smaller than min value! | `survey` |
| `_validation_date` | Constraint | `CHECK (validation_min_date <= validation_max_date)` | Max date cannot be smaller than min date! | `survey` |
| `_validation_datetime` | Constraint | `CHECK (validation_min_datetime <= validation_max_datetime)` | Max datetime cannot be smaller than min datetime! | `survey` |
| `_positive_answer_score` | Constraint | `CHECK (answer_score >= 0)` | An answer score for a non-multiple choice question cannot be negative! | `survey` |
| `_scored_datetime_have_answers` | Constraint | `CHECK (is_scored_question != True OR question_type != 'datetime' OR answer_datetime is not null)` | All "Is a scored question = True" and "Question Type: Datetime" questions need an answer | `survey` |
| `_scored_date_have_answers` | Constraint | `CHECK (is_scored_question != True OR question_type != 'date' OR answer_date is not null)` | All "Is a scored question = True" and "Question Type: Date" questions need an answer | `survey` |
| `_scale` | Constraint | `CHECK (question_type != 'scale' OR (scale_min >= 0 AND scale_max <= 10 AND scale_min < scale_max))` | The scale must be a growing non-empty range between 0 and 10 (inclusive) | `survey` |
| `_is_time_limited_have_time_limit` | Constraint | `CHECK (is_time_limited != TRUE OR time_limit IS NOT NULL AND time_limit > 0)` | All time-limited questions need a positive time limit | `survey` |

## Operations (39)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `survey` | model |  |
| `_check_question_type_for_pages` | validation | self | `survey` | constrains: `is_page` |  |
| `_compute_has_image_only_suggested_answer` | computation | self | `survey` | depends: `suggested_answer_ids`, `suggested_answer_ids.value` |  |
| `_compute_question_placeholder` | computation | self | `survey` | depends: `question_type` |  |
| `_compute_background_image` | computation | self | `survey` | depends: `is_page` | Background image is only available on sections. |
| `_compute_background_image_url` | computation | self | `survey` | depends: `survey_id.access_token`, `background_image`, `page_id`, `survey_id.background_image_url` | How the background url is computed: - For a question: it depends on the related section (see below) - For a section:     - if a section has a background, then we create the background URL using this section's ID     - if not, then we fallback on the survey background url |
| `_compute_question_type` | computation | self | `survey` | depends: `is_page` |  |
| `_compute_question_ids` | computation | self | `survey` | depends: `survey_id.question_and_page_ids.is_page`, `survey_id.question_and_page_ids.sequence` |  |
| `_compute_page_id` | computation | self | `survey` | depends: `survey_id.question_and_page_ids.is_page`, `survey_id.question_and_page_ids.sequence` | Will find the page to which this question belongs to by looking inside the corresponding survey |
| `_compute_save_as_email` | computation | self | `survey` | depends: `question_type`, `validation_email` |  |
| `_compute_save_as_nickname` | computation | self | `survey` | depends: `question_type` |  |
| `_compute_validation_required` | computation | self | `survey` | depends: `question_type` |  |
| `_compute_allowed_triggering_question_ids` | computation | self | `survey` | depends: `survey_id`, `survey_id.question_ids`, `triggering_answer_ids` | Although the question (and possible trigger questions) sequence is used here, we do not add these fields to the dependency list to avoid cascading rpc calls when reordering questions via the webclient. |
| `_compute_triggering_question_ids` | computation | self | `survey` | depends: `triggering_answer_ids` |  |
| `_compute_is_scored_question` | computation | self | `survey` | depends: `question_type`, `scoring_type`, `answer_date`, `answer_datetime`, `answer_numerical_box`, `suggested_answer_ids.is_correct` | Computes whether a question "is scored" or not. Handles following cases: - inconsistent Boolean=None edge case that breaks tests => False - survey is not scored => False - 'date'/'datetime'/'numerical_box' question types w/correct answer => True   (implied without user having to activate, except for numerical whose correct value is 0.0) - 'simple_choice / multiple_choice': set to True if any of suggested answers are marked as correct - question_type isn't scoreable (note: choice questions scoring logic handled separately) => False |
| `_onchange_validation_parameters` | on change | self | `survey` | onchange: `question_type`, `validation_required` | Ensure no value stays set but not visible on form, preventing saving (+consistency with question type). |
| `copy` | lifecycle override | self, default | `survey` |  |  |
| `create` | lifecycle override | self, vals_list | `survey` | model_create_multi |  |
| `_unlink_except_live_sessions_in_progress` | internal rule | self | `survey` | ondelete |  |
| `validate_question` | operation | self, answer, comment | `survey` |  | Validate question, depending on question type and parameters for simple choice, text, date and number, answer is simply the answer of the question. For other multiple choices questions, answer is a list of answers (the selected choices or a list of selected answers per question -for matrix type-):  - Simple answer : `answer = 'example'` or `2` or `question_answer_id` or `2019/10/10` - Multiple choice : `answer = [question_answer_id1, question_answer_id2, question_answer_id3]` - Matrix: `answer = { 'rowId1' : [colId1, colId2,...], 'rowId2' : [colId1, colId3, ...] }`  :returns: A dic |
| `_validate_char_box` | internal rule | self, answer | `survey` |  |  |
| `_validate_numerical_box` | internal rule | self, answer | `survey` |  |  |
| `_validate_date` | internal rule | self, answer | `survey` |  |  |
| `_validate_choice` | internal rule | self, answer, comment | `survey` |  | Validates choice-based questions. - Checks that mandatory questions have at least one answer. - For 'simple_choice', ensures that exactly one answer is provided. |
| `_validate_matrix` | internal rule | self, answers | `survey` |  |  |
| `_validate_scale` | internal rule | self, answer | `survey` |  |  |
| `_index` | internal rule | self | `survey` |  | We would normally just use the 'sequence' field of questions BUT, if the pages and questions are created without ever moving records around, the sequence field can be set to 0 for all the questions.  However, the order of the recordset is always correct so we can rely on the index method. |
| `_update_time_limit_from_survey` | internal rule | self, is_time_limited, time_limit | `survey` |  | Update the speed rating values after a change in survey's speed rating configuration.  * Questions that were not customized will take the new default values from the survey * Questions that were customized will not change their values, but this method will check   and update the `is_time_customized` flag if necessary (to `False`) such that the user   won't need to "actively" do it to make the question sensitive to change in survey values.  This is not done with `_compute`s because `is_time_limited` (and `time_limit`) would depend on `is_time_customized` and vice versa. |
| `_prepare_statistics` | preparation rule | self, user_input_lines | `survey` |  | Compute statistical data for questions by counting number of vote per choice on basis of filter |
| `_get_stats_data` | preparation rule | self, user_input_lines | `survey` |  |  |
| `_get_stats_data_answers` | preparation rule | self, user_input_lines | `survey` |  | Statistics for question.answer based questions (simple choice, multiple choice.). A corner case with a void record survey.question.answer is added to count comments that should be considered as valid answers. This small hack allow to have everything available in the same standard structure. |
| `_get_stats_graph_data_matrix` | preparation rule | self, user_input_lines | `survey` |  |  |
| `_get_stats_data_scale` | preparation rule | self, user_input_lines | `survey` |  |  |
| `_get_stats_summary_data` | preparation rule | self, user_input_lines | `survey` |  |  |
| `_get_stats_summary_data_choice` | preparation rule | self, user_input_lines | `survey` |  |  |
| `_get_stats_summary_data_numerical` | preparation rule | self, user_input_lines, fname | `survey` |  |  |
| `_get_stats_summary_data_scored` | preparation rule | self, user_input_lines | `survey` |  |  |
| `_get_correct_answers` | preparation rule | self | `survey` |  | Return a dictionary linking the scorable question ids to their correct answers. The questions without correct answers are not considered. |
| `_compute_generate_lead` | computation | self | `survey_crm` | depends: `question_type`, `suggested_answer_ids` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_question_type_for_pages` | ValidationError | Question type should be empty for these pages: %s | `survey` |
| `_unlink_except_live_sessions_in_progress` | UserError | You cannot delete questions from surveys "%(survey_names)s" while live sessions are in progress. | `survey` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| `hr_recruitment.group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment_survey` |
| all internal users | no | no | no | no | `survey` |
| `base.group_user` | no | no | no | no | `survey` |
| `group_survey_user` | yes | yes | yes | yes | `survey` |
| `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Survey question: recruitment manager: all recruitment | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey: recruitment interviewer: send surveys to applicants for which they are set as interviewer | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[('survey_id.survey_type', '=', 'recruitment'),                 '\|', ('survey_id.hr_job_ids.interviewer_ids', 'in', user.id),                      ('survey_id.hr_job_ids.application_ids.interviewer_ids', 'in', user.id)                 ]` | 1 | 0 | 0 | 0 |
| Survey question: manager: all | `[(4, ref('group_survey_manager'))]` | `[(1, '=', 1)]` | 1 | 1 | 1 | 1 |
| Survey question: officer: unrestricted survey or in restricted users | `[(4, ref('group_survey_user'))]` | `[                 '\|', ('survey_id.restrict_user_ids', 'in', user.id), ('survey_id.restrict_user_ids', '=', False)]` | 1 | 1 | 1 | 1 |
| Survey question: slide channel officer on certification: read | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[('survey_id.certification', '=', True),             ('survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('survey_id.restrict_user_ids', '=', False),('survey_id.restrict_user_ids', 'in', user.id)]` | 1 | 0 | 0 | 0 |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `survey.survey_question_form` | form |  | `is_placed_before_trigger`, `is_page`, `page_id`, `sequence`, `scoring_type`, `has_image_only_suggested_answer`, `survey_id`, `background_image`, `title`, `questions_selection`, `random_questions_count`, `question_type`, `scale_min`, `scale_max`, `scale_min_label`, `scale_mid_label`, `scale_max_label`, `answer_numerical_box`, `answer_date`, `answer_datetime`, `validation_email`, `save_as_email`, `save_as_nickname`, `is_scored_question`, `answer_score`, `suggested_answer_ids`, `sequence`, `value`, `is_correct`, `answer_score`, `value_image_filename`, `value_image`, `matrix_row_ids`, `sequence`, `value`, `description`, `validation_required`, `validation_length_min`, `validation_min_float_value`, `validation_min_date`, `validation_min_datetime`, `validation_length_max`, `validation_max_float_value`, `validation_max_date`, `validation_max_datetime`, `validation_error_msg`, `matrix_subtype`, `question_placeholder`, `comments_allowed`, `comments_message`, `comment_count_as_answer`, `allowed_triggering_question_ids`, `triggering_answer_ids`, `constr_mandatory`, `constr_error_msg`, `is_time_customized`, `session_available`, `is_time_limited`, `time_limit` |  |  | `survey` |
| `survey.survey_question_tree` | list |  | `title`, `survey_id`, `question_type`, `constr_mandatory` |  |  | `survey` |
| `survey.survey_question_search` | search |  | `title`, `survey_id`, `question_type` |  | `Type`, `Survey` | `survey` |
| `survey_crm.survey_question_view_form` | xpath | `survey.survey_question_form` | `generate_lead` |  |  | `survey_crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `survey.action_survey_question_form` | Questions | list,form | `[('is_page', '=', False)]` | `{'search_default_group_by_page': True, 'show_survey_field': True}` |  | `survey` |

Machine-readable definition: `../../../schemas/data/entities/survey.question.json`; views: `../../../schemas/interfaces/views/survey.question.json`.
