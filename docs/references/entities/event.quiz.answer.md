# Question's Answer (`event.quiz.answer`)

**Transport name:** `event.quiz.answer`  
**Storage name:** `event_quiz_answer`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_track_quiz`

Description: Question's Answer

## Identity and behavior

- Default ordering: `question_id, sequence, id`
- Display name field: `text_value`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  |  |
| `question_id` | Question | many to one | `event.quiz.question` | required; indexed; on delete of the target: cascade |
| `text_value` | Answer | single line text |  | required; translatable |
| `is_correct` | Correct | boolean |  | default  |
| `comment` | Extra Comment | multi line text |  | translatable; Help: This comment will be displayed to the user if they select this answer, after submitting the quiz.                 It is used as a small informational text helping to understand why this answer is correct / incorrect. |
| `awarded_points` | Points | integer |  | default  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_user` | yes | yes | yes | yes | `website_event_track_quiz` |

Machine-readable definition: `../../../schemas/data/entities/event.quiz.answer.json`.
