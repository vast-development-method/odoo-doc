# Event Registration Answer (`event.registration.answer`)

**Transport name:** `event.registration.answer`  
**Storage name:** `event_registration_answer`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `pos_event`

Description: Event Registration Answer

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Display name search fields: `["value_answer_id", "value_text_box"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `question_id` | Question | many to one | `event.question` | required; on delete of the target: restrict; restricted by domain `[('event_ids', 'in', event_id)]` |
| `registration_id` | Registration | many to one | `event.registration` | required; indexed; on delete of the target: cascade |
| `partner_id` | Partner | many to one | `res.partner` | related through path `registration_id.partner_id` |
| `event_id` | Event | many to one | `event.event` | related through path `registration_id.event_id` |
| `question_type` | Question Type | selection |  | related through path `question_id.question_type` |
| `value_answer_id` | Suggested answer | many to one | `event.question.answer` |  |
| `value_text_box` | Text answer | multi line text |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_value_check` | Constraint | `CHECK(value_answer_id IS NOT NULL OR COALESCE(value_text_box, '') <> '')` | There must be a suggested value or a text value. | `event` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `event` | depends: `value_answer_id`, `question_type`, `value_text_box` |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_event` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_event` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | yes | yes | yes | yes | `event` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.event_registration_answer_view_search` | search |  | `value_text_box`, `value_answer_id`, `question_id`, `event_id` |  |  | `event` |
| `event.event_registration_answer_view_tree` | list |  | `registration_id`, `partner_id`, `question_id`, `value_text_box`, `value_answer_id`, `event_id` |  |  | `event` |
| `event.event_registration_answer_view_graph` | graph |  | `value_answer_id`, `event_id` |  |  | `event` |
| `event.event_registration_answer_view_pivot` | pivot |  | `registration_id`, `value_answer_id` |  |  | `event` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event.action_event_registration_report` | Answer Breakdown | list,graph,pivot |  |  |  | `event` |

Machine-readable definition: `../../../schemas/data/entities/event.registration.answer.json`; views: `../../../schemas/interfaces/views/event.registration.answer.json`.
