# Content Quiz Question (`slide.question`)

**Transport name:** `slide.question`  
**Storage name:** `slide_question`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`

Description: Content Quiz Question

## Identity and behavior

- Default ordering: `sequence`
- Display name field: `question`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  |  |
| `question` | Question Name | single line text |  | required; translatable |
| `slide_id` | Content | many to one | `slide.slide` | required; indexed; on delete of the target: cascade |
| `answer_ids` | Answer | one to many | `slide.answer` | inverse field `question_id` |
| `answers_validation_error` | Error on Answers | single line text |  | computed by rule `_compute_answers_validation_error` (not stored) |
| `attempts_count` | Attempts Count | integer |  | computed by rule `_compute_statistics` (not stored); visible only to groups `website_slides.group_website_slides_officer` |
| `attempts_avg` | Attempts Avg | float |  | computed by rule `_compute_statistics` (not stored); visible only to groups `website_slides.group_website_slides_officer`; precision `[6, 2]` |
| `done_count` | Done Count | integer |  | computed by rule `_compute_statistics` (not stored); visible only to groups `website_slides.group_website_slides_officer` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_answers_integrity` | validation | self | `website_slides` | constrains: `answer_ids` |  |
| `_compute_statistics` | computation | self | `website_slides` | depends: `slide_id` |  |
| `_compute_answers_validation_error` | computation | self | `website_slides` | depends: `answer_ids`, `answer_ids.is_correct` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_answers_integrity` | ValidationError | All questions must have at least one correct answer and one incorrect answer:  %s | `website_slides` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_slides` |
| `base.group_portal` | no | yes | no | no | `website_slides` |
| `base.group_user` | no | yes | no | no | `website_slides` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_question_view_form` | form |  | `answers_validation_error`, `question`, `answer_ids`, `display_name`, `text_value`, `is_correct`, `comment` |  |  | `website_slides` |
| `website_slides.slide_question_view_tree` | list |  | `sequence`, `question`, `slide_id` |  |  | `website_slides` |
| `website_slides.slide_question_view_tree_report` | list |  | `sequence`, `question`, `slide_id`, `attempts_count`, `attempts_avg`, `done_count` |  |  | `website_slides` |
| `website_slides.slide_question_view_search` | search |  | `question`, `slide_id` |  |  | `website_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_question_action_report` | Quizzes | list,graph,pivot,form |  |  |  | `website_slides` |

Machine-readable definition: `../../../schemas/data/entities/slide.question.json`; views: `../../../schemas/interfaces/views/slide.question.json`.
