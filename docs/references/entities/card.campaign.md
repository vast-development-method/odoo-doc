# Marketing Card Campaign (`card.campaign`)

**Transport name:** `card.campaign`  
**Storage name:** `card_campaign`  
**Kind:** persistent entity (one table)  
**Defined by package:** `marketing_card`

Description: Marketing Card Campaign

## Identity and behavior

- Mixins (classical inheritance): `mail.activity.mixin`, `mail.render.mixin`, `mail.thread`
- Default ordering: `id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (44)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `active` | Active | boolean |  | default `True` |
| `body_html` | Body Hypertext markup language | rich text |  | related through path `card_template_id.body` |
| `card_count` | Card Count | integer |  | computed by rule `_compute_card_stats` (not stored) |
| `card_click_count` | Card Click Count | integer |  | computed by rule `_compute_card_stats` (not stored) |
| `card_share_count` | Card Share Count | integer |  | computed by rule `_compute_card_stats` (not stored) |
| `mailing_ids` | Mailing | one to many | `mailing.mailing` | inverse field `card_campaign_id` |
| `mailing_count` | Mailing Count | integer |  | computed by rule `_compute_mailing_count` (not stored) |
| `card_ids` | Card | one to many | `card.card` | inverse field `campaign_id` |
| `card_template_id` | Design | many to one | `card.template` | required; default computed dynamically (_default_card_template_id) |
| `image_preview` | Image Preview | image |  | read only; computed by rule `_compute_image_preview` and stored |
| `link_tracker_id` | Link Tracker | many to one | `link.tracker` | on delete of the target: restrict |
| `res_model` | Model Name | selection |  | required; read only; computed by rule `_compute_res_model` and stored; values provided by rule `_get_model_selection`; precomputed before insertion |
| `post_suggestion` | Post Suggestion | multi line text |  | Help: Description below the card and default text when sharing on X |
| `preview_record_ref` | Preview On | reference |  | required; values provided by rule `_get_model_selection` |
| `tag_ids` | Tags | many to many | `card.campaign.tag` |  |
| `target_url` | Post Link | single line text |  |  |
| `target_url_click_count` | Target Uniform resource locator Click Count | integer |  | related through path `link_tracker_id.count` |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); restricted by domain `[('share', '=', False)]` |
| `reward_message` | Thank You Message | rich text |  |  |
| `reward_target_url` | Reward Link | single line text |  |  |
| `request_title` | Request | single line text |  | default computed dynamically (lambda self: _('Help us share the news')) |
| `request_description` | Request Description | multi line text |  |  |
| `content_background` | Background | image |  |  |
| `content_button` | Button | single line text |  |  |
| `content_header` | Header | single line text |  |  |
| `content_header_dyn` | Is Dynamic Header | boolean |  |  |
| `content_header_path` | Header Path | single line text |  |  |
| `content_header_color` | Header Color | single line text |  |  |
| `content_sub_header` | Sub-Header | single line text |  |  |
| `content_sub_header_dyn` | Is Dynamic Sub-Header | boolean |  |  |
| `content_sub_header_path` | Sub-Header Path | single line text |  |  |
| `content_sub_header_color` | Sub Header Color | single line text |  |  |
| `content_section` | Section | single line text |  |  |
| `content_section_dyn` | Is Dynamic Section | boolean |  |  |
| `content_section_path` | Section Path | single line text |  |  |
| `content_sub_section1` | Sub-Section 1 | single line text |  |  |
| `content_sub_section1_dyn` | Is Dynamic Sub-Section 1 | boolean |  |  |
| `content_sub_section1_path` | Sub-Section 1 Path | single line text |  |  |
| `content_sub_section2` | Sub-Section 2 | single line text |  |  |
| `content_sub_section2_dyn` | Is Dynamic Sub-Section 2 | boolean |  |  |
| `content_sub_section2_path` | Sub-Section 2 Path | single line text |  |  |
| `content_image1_path` | Dynamic Image 1 | single line text |  |  |
| `content_image2_path` | Dynamic Image 2 | single line text |  |  |

## Operations (23)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_card_template_id` | preparation rule | self | `marketing_card` |  |  |
| `_get_model_selection` | preparation rule | self | `marketing_card` |  | Hardcoded list of models, checked against actually-present models. |
| `_compute_card_stats` | computation | self | `marketing_card` | depends: `card_ids` |  |
| `_get_render_fields` | preparation rule | self | `marketing_card` | model |  |
| `_check_access_right_dynamic_template` | validation | self | `marketing_card` |  | `_unrestricted_rendering` being True means we trust the value on model when rendering. This means once created, rendering is done without restriction. But this attribute triggers a check at create / write / translation update that current user is an admin or has full edition rights (group_mail_template_editor).   However here a Marketing Card Manager must be able to edit the fields other  than the rendering fields. The qweb rendered field `body_html` cannot be  modified by users other than the `base.group_system` users, as - it's a related field to `card.template.body`, - store=False - the mod |
| `_compute_image_preview` | computation | self | `marketing_card` | depends: |  |
| `_compute_mailing_count` | computation | self | `marketing_card` | depends: `mailing_ids` |  |
| `_compute_res_model` | computation | self | `marketing_card` | depends: `preview_record_ref` |  |
| `create` | lifecycle override | self, vals_list | `marketing_card` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `marketing_card` |  |  |
| `action_view_cards` | user action | self | `marketing_card` |  |  |
| `action_view_cards_clicked` | user action | self | `marketing_card` |  |  |
| `action_view_cards_shared` | user action | self | `marketing_card` |  |  |
| `action_view_mailings` | user action | self | `marketing_card` |  |  |
| `action_preview` | user action | self | `marketing_card` |  |  |
| `action_share` | user action | self | `marketing_card` |  |  |
| `_fetch_or_create_preview_card` | internal rule | self | `marketing_card` |  | Fetch the card corresponding to the preview record, or create one if none exists.  The image also gets the preview render if it has none. It is also archived to ensure it is rerendered later if sent. |
| `_action_share_get_default_body` | internal rule | self | `marketing_card` |  |  |
| `_get_image_b64` | preparation rule | self, record | `marketing_card` |  |  |
| `_update_cards` | internal rule | self, domain, auto_commit | `marketing_card` |  | Create missing cards and update cards if necessary based for the domain. |
| `_get_url_from_res_id` | preparation rule | self, res_id, suffix | `marketing_card` |  |  |
| `_compute_render_model` | computation | self | `marketing_card` | depends: `res_model` | override for mail.render.mixin |
| `_get_card_element_values` | preparation rule | self, record | `marketing_card` |  | Helper to get the right value for dynamic fields. |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | ValidationError | Model of campaign %(campaign)s may not be changed as it already has cards | `marketing_card` |
| `_get_image_b64` | UserError | An error occured while rendering a card for %(record_name)s. Try again or check the server logs for more details. | `marketing_card` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `marketing_card.marketing_card_group_user` | yes | yes | yes | yes | `marketing_card` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Manager may access and edit any card campaign | `[(4, ref('marketing_card_group_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Users may only edit their own card campaigns | `[(4, ref('marketing_card_group_user'))]` | `[('user_id', '=', user.id)]` | False | True | False | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `marketing_card.card_campaign_view_form` | form |  | `mailing_count`, `target_url_click_count`, `card_count`, `card_click_count`, `card_share_count`, `name`, `preview_record_ref`, `target_url`, `res_model`, `res_model`, `post_suggestion`, `user_id`, `tag_ids`, `content_background`, `content_header_dyn`, `content_header`, `content_header_path`, `content_header_color`, `content_sub_header_dyn`, `content_sub_header`, `content_sub_header_path`, `content_sub_header_color`, `content_section_dyn`, `content_section`, `content_section_path`, `content_sub_section1_dyn`, `content_sub_section1`, `content_sub_section1_path`, `content_sub_section2_dyn`, `content_sub_section2`, `content_sub_section2_path`, `content_image1_path`, `content_image2_path`, `content_button`, `card_template_id`, `image_preview`, `request_title`, `request_description`, `reward_target_url`, `reward_message` | `action_share`, `action_preview`, `action_view_mailings`, , `action_view_cards`, `action_view_cards_clicked`, `action_view_cards_shared` |  | `marketing_card` |
| `marketing_card.card_campaign_view_kanban` | kanban |  | `name`, `tag_ids`, `card_share_count`, `target_url_click_count`, `user_id` |  |  | `marketing_card` |
| `marketing_card.card_campaign_view_tree` | list |  | `create_date`, `name`, `user_id`, `res_model`, `target_url`, `tag_ids` |  |  | `marketing_card` |
| `marketing_card.card_campaign_view_search` | search |  | `name`, `tag_ids` |  | `My Campaigns`, `Archived`, `Responsible`, `Tags` | `marketing_card` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `marketing_card.card_campaign_action` | Card Campaign | list,kanban,form |  |  |  | `marketing_card` |

Machine-readable definition: `../../../schemas/data/entities/card.campaign.json`; views: `../../../schemas/interfaces/views/card.campaign.json`.
