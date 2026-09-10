# Survey Label (`survey.question.answer`)

**Transport name:** `survey.question.answer`  
**Storage name:** `survey_question_answer`  
**Kind:** persistent entity (one table)  
**Defined by package:** `survey`  
**Extended by packages:** `survey_crm`

Description: Survey Label

## Identity and behavior

- Default ordering: `question_id, sequence, id`
- Display name field: `value`
- Display name search fields: `["question_id.title", "value"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `question_id` | Question | many to one | `survey.question` | indexed (btree_not_null); on delete of the target: cascade |
| `matrix_question_id` | Question (as matrix row) | many to one | `survey.question` | indexed (btree_not_null); on delete of the target: cascade |
| `question_type` | Question Type | selection |  | related through path `question_id.question_type` |
| `sequence` | Label Sequence order | integer |  | default `10` |
| `scoring_type` | Scoring Type | selection |  | related through path `question_id.scoring_type` |
| `value` | Suggested value | single line text |  | translatable |
| `value_image` | Image | image |  |  |
| `value_image_filename` | Image Filename | single line text |  |  |
| `value_label` | Value Label | single line text |  | computed by rule `_compute_value_label` (not stored); Help: Answer label as either the value itself if not empty or a letter representing the index of the answer otherwise. |
| `is_correct` | Correct | boolean |  |  |
| `answer_score` | Score | float |  | Help: A positive score indicates a correct choice; a negative or null score indicates a wrong answer |
| `generate_lead` | Lead creation | boolean |  | Help: Creates a lead when participants choose this answer |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_value_not_empty` | Constraint | `CHECK (value IS NOT NULL OR value_image_filename IS NOT NULL)` | Suggested answer value must not be empty (a text and/or an image must be provided). | `survey` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `survey` | depends: `value_label`, `question_id.question_type`, `question_id.title`, `matrix_question_id` | Render an answer name as "Question title : Answer label value", making sure it is not too long.  Unless the answer is part of a matrix-type question, this implementation makes sure we have at least 30 characters for the question title, then we elide it, leaving the rest of the space for the answer. |
| `_compute_value_label` | computation | self | `survey` | depends: `question_id.suggested_answer_ids`, `sequence`, `value` | Compute the label as the value if not empty or a letter representing the index of the answer otherwise. |
| `_check_question_not_empty` | validation | self | `survey` | constrains: `question_id`, `matrix_question_id` | Ensure that field question_id XOR field matrix_question_id is not null |
| `_get_answer_matching_domain` | preparation rule | self, row_id | `survey` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_question_not_empty` | ValidationError | A label must be attached to only one question. | `survey` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| all internal users | no | no | no | no | `survey` |
| `base.group_user` | no | no | no | no | `survey` |
| `group_survey_user` | yes | yes | yes | yes | `survey` |
| `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Survey question answer: recruitment manager: all recruitment | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `['\|', ('question_id.survey_id.survey_type', '=', 'recruitment'),                 ('matrix_question_id.survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey question answer: manager: all | `[(4, ref('group_survey_manager'))]` | `[(1, '=', 1)]` | 1 | 1 | 1 | 1 |
| Survey question answer: officer: unrestricted survey or in restricted users | `[(4, ref('group_survey_user'))]` | `[                 '\|',                     '\|', ('question_id.survey_id.restrict_user_ids', 'in', user.id), ('matrix_question_id.survey_id.restrict_user_ids', 'in', user.id),                     '\|', ('question_id.survey_id.restrict_user_ids', '=', False), ('matrix_question_id.survey_id.restrict_user_ids', '=', False)]` | 1 | 1 | 1 | 1 |
| Survey question answer: slide channel officer on certification: read | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[             '\|',                 '&',                     '&', ('question_id.survey_id.certification', '=', True), ('question_id.survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),                     '\|', ('question_id.survey_id.restrict_user_ids', '=', False), ('question_id.survey_id.restrict_user_ids', 'in', user.id),                 '&',                     '&', ('matrix_question_id.survey_id.certification', '=', True), ('matrix_question_id.survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),                     '\|', ('matrix_question_id.survey_id.restrict_user_ids', '=', False), ('matrix_question_id.survey_id.restrict_user_ids', 'in', user.id),         ]` | 1 | 0 | 0 | 0 |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `survey.survey_question_answer_view_tree` | list |  | `sequence`, `question_id`, `value`, `answer_score` |  |  | `survey` |
| `survey.survey_question_answer_view_form` | form |  | `question_type`, `scoring_type`, `question_id`, `is_correct`, `answer_score`, `value_image`, `value`, `matrix_question_id`, `sequence` |  |  | `survey` |
| `survey.survey_question_answer_view_search` | search |  | `question_id` |  | `Question` | `survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `survey.survey_question_answer_action` | Suggested Values | list,form |  | `{'search_default_group_by_question': True}` |  | `survey` |

Machine-readable definition: `../../../schemas/data/entities/survey.question.answer.json`; views: `../../../schemas/interfaces/views/survey.question.answer.json`.
