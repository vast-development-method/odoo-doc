# Slide Question's Answer (`slide.answer`)

**Transport name:** `slide.answer`  
**Storage name:** `slide_answer`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`

Description: Slide Question's Answer

## Identity and behavior

- Default ordering: `question_id, sequence, id`
- Display name field: `text_value`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  |  |
| `question_id` | Question | many to one | `slide.question` | required; indexed; on delete of the target: cascade |
| `text_value` | Answer | single line text |  | required; translatable |
| `is_correct` | Is correct answer | boolean |  |  |
| `comment` | Comment | multi line text |  | translatable; Help: This comment will be displayed to the user if they select this answer |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `website_slides` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |

Machine-readable definition: `../../../schemas/data/entities/slide.answer.json`.
