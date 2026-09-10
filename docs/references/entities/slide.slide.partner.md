# Slide / Partner decorated m2m (`slide.slide.partner`)

**Transport name:** `slide.slide.partner`  
**Storage name:** `slide_slide_partner`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`  
**Extended by packages:** `website_slides_survey`

Description: Slide / Partner decorated m2m

## Identity and behavior

- Display name field: `partner_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `slide_id` | Content | many to one | `slide.slide` | required; indexed; on delete of the target: cascade |
| `slide_category` | Slide Category | selection |  | related through path `slide_id.slide_category` |
| `channel_id` | Channel | many to one | `slide.channel` | related through path `slide_id.channel_id` and stored; indexed; on delete of the target: cascade |
| `partner_id` | Partner | many to one | `res.partner` | required; indexed; on delete of the target: cascade |
| `vote` | Vote | integer |  | default  |
| `completed` | Completed | boolean |  |  |
| `quiz_attempts_count` | Quiz attempts count | integer |  | default  |
| `user_input_ids` | Certification attempts | one to many | `survey.user_input` | inverse field `slide_partner_id` |
| `survey_scoring_success` | Certification Succeeded | boolean |  | computed by rule `_compute_survey_scoring_success` and stored |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_slide_partner_uniq` | Constraint | `unique(slide_id, partner_id)` | A partner membership to a slide must be unique! | `website_slides` |
| `_check_vote` | Constraint | `CHECK(vote IN (-1, 0, 1))` | The vote must be 1, 0 or -1. | `website_slides` |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `website_slides` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website_slides` |  |  |
| `_recompute_completion` | internal rule | self | `website_slides_survey`, `website_slides` |  |  |
| `_compute_survey_scoring_success` | computation | self | `website_slides_survey` | depends: `partner_id`, `user_input_ids.scoring_success` |  |
| `_compute_field_value` | computation | self, field | `website_slides_survey` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `website_slides` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Slide Partner: officer: create/write/unlink own only | `[(4, ref('group_website_slides_officer'))]` | `[('channel_id.user_id', '=', user.id)]` | 0 | 1 | 1 | 1 |
| Slide Partner: manager: crud all | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | 1 | 1 | 1 | 1 |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_slide_partner_view_search` | search |  | `partner_id`, `slide_id`, `channel_id` |  | `Completed`, `Content` | `website_slides` |
| `website_slides.slide_slide_partner_view_tree` | list |  | `create_date`, `partner_id`, `slide_id`, `channel_id`, `completed`, `quiz_attempts_count`, `vote` |  |  | `website_slides` |
| `website_slides.slide_slide_partner_view_form` | form |  | `partner_id`, `slide_id`, `slide_category`, `channel_id`, `completed`, `quiz_attempts_count`, `vote` |  |  | `website_slides` |
| `website_slides_survey.slide_slide_partner_view_search` | xpath | `website_slides.slide_slide_partner_view_search` |  |  | `Certification Passed` | `website_slides_survey` |
| `website_slides_survey.slide_slide_partner_view_tree` | xpath | `website_slides.slide_slide_partner_view_tree` | `survey_scoring_success` |  |  | `website_slides_survey` |
| `website_slides_survey.slide_slide_partner_view_form` | xpath | `website_slides.slide_slide_partner_view_form` | `survey_scoring_success` |  |  | `website_slides_survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_slide_partner_action_from_slide` | Attendees | list,form,kanban | `[('slide_id', '=', active_id)]` | `{'default_slide_id': active_id}` |  | `website_slides` |

Machine-readable definition: `../../../schemas/data/entities/slide.slide.partner.json`; views: `../../../schemas/interfaces/views/slide.slide.partner.json`.
