# Survey User Input Line (`survey.user_input.line`)

**Transport name:** `survey.user_input.line`  
**Storage name:** `survey_user_input_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `survey`

Description: Survey User Input Line

## Identity and behavior

- Default ordering: `question_sequence, id`
- Display name field: `user_input_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_input_id` | User Input | many to one | `survey.user_input` | required; indexed; on delete of the target: cascade |
| `survey_id` | Survey | many to one |  | related through path `user_input_id.survey_id` and stored |
| `question_id` | Question | many to one | `survey.question` | required; indexed; on delete of the target: cascade |
| `page_id` | Section | many to one |  | related through path `question_id.page_id` |
| `question_sequence` | Sequence | integer |  | related through path `question_id.sequence` and stored |
| `lang_id` | Lang | many to one | `res.lang` | related through path `user_input_id.lang_id` |
| `skipped` | Skipped | boolean |  |  |
| `answer_type` | Answer Type | selection |  |  |
| `value_char_box` | Text answer | single line text |  |  |
| `value_numerical_box` | Numerical answer | float |  |  |
| `value_scale` | Scale value | integer |  |  |
| `value_date` | Date answer | date |  |  |
| `value_datetime` | Datetime answer | date and time |  |  |
| `value_text_box` | Free Text answer | multi line text |  |  |
| `suggested_answer_id` | Suggested answer | many to one | `survey.question.answer` |  |
| `matrix_row_id` | Row answer | many to one | `survey.question.answer` |  |
| `answer_score` | Score | float |  | computed by rule `_compute_answer_score` and stored; precomputed before insertion |
| `answer_is_correct` | Correct | boolean |  | computed by rule `_compute_answer_score` and stored; precomputed before insertion |

## Selection values

### `answer_type` (Answer Type)

| Value | Label |
|---|---|
| `text_box` | Free Text |
| `char_box` | Text |
| `numerical_box` | Number |
| `scale` | Number |
| `date` | Date |
| `datetime` | Datetime |
| `suggestion` | Suggestion |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `survey` | depends: `answer_type`, `value_text_box`, `value_numerical_box`, `value_char_box`, `value_date`, `value_datetime`, `suggested_answer_id.value`, `matrix_row_id.value` |  |
| `_compute_answer_score` | computation | self | `survey` | depends: `answer_type`, `value_text_box`, `value_numerical_box`, `value_date`, `value_datetime`, `suggested_answer_id`, `user_input_id` | Get values for: answer_is_correct and associated answer_score.  Calculates whether an answer_is_correct and its score based on 'answer_type' and corresponding question. Handles choice (answer_type == 'suggestion') questions separately from other question types. Each selected choice answer is handled as an individual answer.  If score depends on the speed of the answer, it is adjusted as follows:  - If the user answers in less than 2 seconds, they receive 100% of the possible points.  - If user answers after that, they receive 50% of the possible points + the remaining     50% scaled by the tim |
| `_check_answer_type_skipped` | validation | self | `survey` | constrains: `skipped`, `answer_type` |  |
| `_get_answer_matching_domain` | preparation rule | self | `survey` |  |  |
| `_get_answer_value` | preparation rule | self | `survey` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_answer_type_skipped` | ValidationError | A question can either be skipped or answered, not both. | `survey` |
| `_check_answer_type_skipped` | ValidationError | The answer must be in the right type | `survey` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| `hr_recruitment.group_hr_recruitment_user` | no | yes | no | no | `hr_recruitment_survey` |
| `hr_recruitment.group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment_survey` |
| all internal users | no | no | no | no | `survey` |
| `base.group_user` | no | no | no | no | `survey` |
| `group_survey_user` | no | yes | no | no | `survey` |
| `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Survey user input line: recruitment manager: all recruitment | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey user input line: recruitment officer: unrestricted or in restricted users | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[                 '&', ('survey_id.survey_type', '=', 'recruitment'),                 '\|',  ('survey_id.restrict_user_ids', 'in', user.id),                         ('survey_id.restrict_user_ids', '=', False)]` | 1 | 0 | 0 | 0 |
| Survey user input line: recruitment interviewer: read survey answers for which they are set as interviewer | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[                 '\|',                     ('user_input_id.applicant_id.interviewer_ids', 'in', user.id),                     ('user_input_id.applicant_id.job_id.interviewer_ids', 'in', user.id),                 ]` | 1 | 0 | 0 | 0 |
| Survey user input line: manager: all non specialized surveys | `[(4, ref('group_survey_manager'))]` | `[('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey'))]` | 1 | 1 | 1 | 1 |
| Survey user input line: officer: unrestricted survey or in restricted users | `[(4, ref('group_survey_user'))]` | `[                 '&', ('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey')),                 '\|', ('survey_id.restrict_user_ids', 'in', user.id),                      ('survey_id.restrict_user_ids', '=', False)]` | 1 | 1 | 1 | 1 |
| Survey user input line: slide channel officer on certification: read | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[('survey_id.certification', '=', True),             ('survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('survey_id.restrict_user_ids', '=', False), ('survey_id.restrict_user_ids', 'in', user.id)]` | 1 | 0 | 0 | 0 |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `survey.survey_user_input_line_view_form` | form |  | `question_id`, `create_date`, `answer_type`, `skipped`, `answer_score`, `value_char_box`, `value_numerical_box`, `value_date`, `value_datetime`, `value_text_box`, `matrix_row_id`, `suggested_answer_id` |  |  | `survey` |
| `survey.survey_response_line_view_tree` | list |  | `survey_id`, `user_input_id`, `question_id`, `create_date`, `answer_type`, `skipped`, `answer_score` |  |  | `survey` |
| `survey.survey_user_input_line_view_search` | search |  | `user_input_id`, `survey_id` |  | `Survey`, `Language`, `User Input` | `survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `survey.survey_user_input_line_action` | Detailed Answers | list,form | `[('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey'))]` | `{'search_default_group_by_survey': True, 'search_default_group_by_user_input': True}` |  | `survey` |

Machine-readable definition: `../../../schemas/data/entities/survey.user_input.line.json`; views: `../../../schemas/interfaces/views/survey.user_input.line.json`.
