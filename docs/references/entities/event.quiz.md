# Quiz (`event.quiz`)

**Transport name:** `event.quiz`  
**Storage name:** `event_quiz`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_track_quiz`

Description: Quiz

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `question_ids` | Questions | one to many | `event.quiz.question` | inverse field `quiz_id` |
| `event_track_id` | Event Track | many to one | `event.track` | read only; indexed (btree_not_null) |
| `event_id` | Event | many to one | `event.event` | read only; related through path `event_track_id.event_id` and stored |
| `repeatable` | Unlimited Tries | boolean |  | Help: Let attendees reset the quiz and try again. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_user` | yes | yes | yes | yes | `website_event_track_quiz` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_track_quiz.event_quiz_view_search` | search |  | `name`, `event_track_id`, `event_id` |  | `Track`, `Event` | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_view_tree` | list |  | `name`, `event_id`, `event_track_id` |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_view_form` | form |  | `name`, `repeatable`, `event_id`, `event_track_id`, `question_ids` |  |  | `website_event_track_quiz` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_track_quiz.event_quiz_action` | Event Quizzes | list,form |  | `{'create': False}` |  | `website_event_track_quiz` |

Machine-readable definition: `../../../schemas/data/entities/event.quiz.json`; views: `../../../schemas/interfaces/views/event.quiz.json`.
