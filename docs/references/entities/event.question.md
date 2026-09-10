# Event Question (`event.question`)

**Transport name:** `event.question`  
**Storage name:** `event_question`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `pos_event`

Description: Event Question

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence,id`
- Display name field: `title`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `title` | Title | single line text |  | required; translatable |
| `question_type` | Question Type | selection |  | required; default `simple_choice` |
| `active` | Active | boolean |  | default `True` |
| `event_type_ids` | Event Types | many to many | `event.type` | not copied on duplication |
| `event_ids` | Events | many to many | `event.event` | not copied on duplication |
| `event_count` | # Events | integer |  | computed by rule `_compute_event_count` (not stored) |
| `is_default` | Default question | boolean |  | Help: Include by default in new events. |
| `is_reusable` | Is Reusable | boolean |  | computed by rule `_compute_is_reusable` and stored; default `True`; Help: Allow this question to be selected and reused for any future event. Always true for default questions. |
| `answer_ids` | Answers | one to many | `event.question.answer` | inverse field `question_id` |
| `sequence` | Sequence | integer |  | default `10` |
| `once_per_order` | Ask once per order | boolean |  | Help: Check this for order-level questions (e.g., 'Company Name') where the answer is the same for everyone. |
| `is_mandatory_answer` | Mandatory Answer | boolean |  |  |

## Selection values

### `question_type` (Question Type)

| Value | Label |
|---|---|
| `simple_choice` | Selection |
| `text_box` | Text Input |
| `name` | Name |
| `email` | Email |
| `phone` | Phone |
| `company_name` | Company |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_default_question_is_reusable` | Constraint | `CHECK(is_default IS DISTINCT FROM TRUE OR is_reusable IS TRUE)` | A default question must be reusable. | `event` |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_event_count` | computation | self | `event` | depends: `event_ids` |  |
| `_compute_is_reusable` | computation | self | `event` | depends: `is_default`, `event_type_ids` |  |
| `write` | lifecycle override | self, vals | `event` |  | We add a check to prevent changing the question_type of a question that already has answers. Indeed, it would mess up the event.registration.answer (answer type not matching the question type). |
| `_unlink_except_answered_question` | internal rule | self | `event` | ondelete |  |
| `_unlink_except_default_question` | internal rule | self | `event` | ondelete |  |
| `action_view_question_answers` | user action | self | `event` |  | Allow analyzing the attendees answers to event questions in a convenient way:  - A graph view showing counts of each suggestion for simple_choice questions   (Along with secondary pivot and list views) - A list view showing textual answers values for text_box questions. |
| `action_event_view` | user action | self | `event` |  |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_event` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_event` | model |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | You cannot change the question type of a question that already has answers! | `event` |
| `_unlink_except_answered_question` | UserError | You cannot delete a question that has already been answered by attendees. You can archive it instead. | `event` |
| `_unlink_except_default_question` | UserError | You cannot delete a default question. | `event` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.group_event_user` | yes | yes | yes | yes | `event` |
| `base.group_public` | no | yes | no | no | `website_event` |
| `base.group_portal` | no | yes | no | no | `website_event` |
| `base.group_user` | no | yes | no | no | `website_event` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Question: not event groups: event published read | `[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('event_ids', 'any', [('is_published', '=', True)])]` | True | False | False | False |
| Event Question: event user: read all | `[(4, ref('event.group_event_registration_desk'))]` | `[(1, '=', 1)]` | True | False | False | False |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.event_question_view_search` | search |  | `title`, `question_type`, `answer_ids` |  | `Mandatory`, `Not mandatory`, `Once per order`, `For each attendee`, `Reusable`, `Not Reusable`, `Default Questions`, `Archived`, `Event` | `event` |
| `event.event_question_view_form` | form |  | `event_count`, `title`, `is_mandatory_answer`, `question_type`, `once_per_order`, `is_default`, `event_type_ids`, `is_reusable`, `answer_ids`, `display_name`, `sequence`, `name` | `action_view_question_answers`, `action_event_view` |  | `event` |
| `event.event_question_view_list` | list |  | `sequence`, `title`, `is_mandatory_answer`, `once_per_order`, `question_type`, `answer_ids`, `is_default`, `is_reusable` | `Stats` |  | `event` |
| `event.event_question_view_list_add` | xpath | `event.event_question_view_list` |  |  |  | `event` |
| `event_crm.event_question_view_form` | xpath | `event.event_question_view_form` |  | `Add a rule` |  | `event_crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event.event_question_action` | Event Question | list,form |  |  |  | `event` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `event.event_question_menu` |  |  | `event.event_question_action` |  |  |

Machine-readable definition: `../../../schemas/data/entities/event.question.json`; views: `../../../schemas/interfaces/views/event.question.json`.
