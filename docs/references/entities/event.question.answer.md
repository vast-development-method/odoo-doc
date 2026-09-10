# Event Question Answer (`event.question.answer`)

**Transport name:** `event.question.answer`  
**Storage name:** `event_question_answer`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `event_crm`, `pos_event`

Description: Event Question Answer

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence,id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Answer | single line text |  | required; translatable |
| `question_id` | Question | many to one | `event.question` | required; indexed; on delete of the target: cascade |
| `sequence` | Sequence | integer |  | default `10` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_unlink_except_selected_answer` | internal rule | self | `event` | ondelete |  |
| `action_add_rule_button` | user action | self | `event_crm` |  |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_event` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_event` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_selected_answer` | UserError | You cannot delete an answer that has already been selected by attendees. | `event` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_user` | yes | yes | yes | yes | `event` |
| `event.group_event_registration_desk` | no | yes | yes | no | `event` |
| `event.group_event_user` | yes | yes | yes | yes | `event` |
| `base.group_public` | no | yes | no | no | `website_event` |
| `base.group_portal` | no | yes | no | no | `website_event` |
| `base.group_user` | no | yes | no | no | `website_event` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Question Answer: not event groups: event published read | `[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('question_id.event_ids', 'any', [('is_published', '=', True)])]` | True | False | False | False |
| Event Question Answer: event user: read all | `[(4, ref('event.group_event_registration_desk'))]` | `[(1, '=', 1)]` | True | False | False | False |

Machine-readable definition: `../../../schemas/data/entities/event.question.answer.json`.
