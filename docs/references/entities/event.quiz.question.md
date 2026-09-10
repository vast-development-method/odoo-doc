# Content Quiz Question (`event.quiz.question`)

**Transport name:** `event.quiz.question`  
**Storage name:** `event_quiz_question`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_track_quiz`

Description: Content Quiz Question

## Identity and behavior

- Default ordering: `quiz_id, sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Question | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  |  |
| `quiz_id` | Quiz | many to one | `event.quiz` | required; indexed; on delete of the target: cascade |
| `correct_answer_id` | Correct Answer | one to many | `event.quiz.answer` | computed by rule `_compute_correct_answer_id` (not stored) |
| `awarded_points` | Number of Points | integer |  | computed by rule `_compute_awarded_points` (not stored) |
| `answer_ids` | Answer | one to many | `event.quiz.answer` | inverse field `question_id` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_awarded_points` | computation | self | `website_event_track_quiz` | depends: `answer_ids.awarded_points` |  |
| `_compute_correct_answer_id` | computation | self | `website_event_track_quiz` | depends: `answer_ids.is_correct` |  |
| `_check_answers_integrity` | validation | self | `website_event_track_quiz` | constrains: `answer_ids` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_answers_integrity` | ValidationError | Question "%s" must have 1 correct answer to be valid. | `website_event_track_quiz` |
| `_check_answers_integrity` | ValidationError | Question "%s" must have 1 correct answer and at least 1 incorrect answer to be valid. | `website_event_track_quiz` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_user` | yes | yes | yes | yes | `website_event_track_quiz` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_track_quiz.event_quiz_question_view_search` | search |  | `name`, `quiz_id` |  | `Quiz` | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_view_tree` | list |  | `sequence`, `name`, `quiz_id`, `awarded_points` |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_view_tree_from_quiz` | xpath | `website_event_track_quiz.event_quiz_question_view_tree` |  |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_view_form` | form |  | `name`, `quiz_id`, `awarded_points`, `answer_ids`, `sequence`, `text_value`, `is_correct`, `awarded_points`, `comment` |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_view_form_from_quiz` | xpath | `website_event_track_quiz.event_quiz_question_view_form` |  |  |  | `website_event_track_quiz` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_track_quiz.event_quiz_question_action` | Event Quiz Questions | list,form |  | `{'create': False}` |  | `website_event_track_quiz` |

Machine-readable definition: `../../../schemas/data/entities/event.quiz.question.json`; views: `../../../schemas/interfaces/views/event.quiz.question.json`.
