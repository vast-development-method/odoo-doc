# Chatbot Script (`chatbot.script`)

**Transport name:** `chatbot.script`  
**Storage name:** `chatbot_script`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`  
**Extended by packages:** `crm_livechat`, `website_livechat`

Description: Chatbot Script

## Identity and behavior

- Mixins (classical inheritance): `image.mixin`, `utm.source.mixin`
- Default ordering: `title, id`
- Display name field: `title`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `title` | Title | single line text |  | required; default `Chatbot`; translatable |
| `active` | Active | boolean |  | default `True` |
| `image_1920` | Image 1920 | image |  | related through path `operator_partner_id.image_1920` |
| `script_step_ids` | Script Steps | one to many | `chatbot.script.step` | inverse field `chatbot_script_id` |
| `operator_partner_id` | Bot Operator | many to one | `res.partner` | required; indexed; not copied on duplication; on delete of the target: restrict |
| `livechat_channel_count` | Livechat Channel Count | integer |  | computed by rule `_compute_livechat_channel_count` (not stored) |
| `first_step_warning` | First Step Warning | selection |  | computed by rule `_compute_first_step_warning` (not stored) |
| `lead_count` | Generated Lead Count | integer |  | computed by rule `_compute_lead_count` (not stored) |

## Selection values

### `first_step_warning` (First Step Warning)

| Value | Label |
|---|---|
| `first_step_operator` | First Step Operator |
| `first_step_invalid` | First Step Invalid |

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_question_selection` | validation | self | `im_livechat` | constrains: `script_step_ids` |  |
| `_onchange_script_step_ids` | on change | self | `im_livechat` | onchange: `script_step_ids` |  |
| `_compute_livechat_channel_count` | computation | self | `im_livechat` |  |  |
| `_compute_first_step_warning` | computation | self | `im_livechat` | depends: `script_step_ids.is_forward_operator`, `script_step_ids.step_type` |  |
| `copy_data` | lifecycle override | self, default | `im_livechat` |  |  |
| `copy` | lifecycle override | self, default | `im_livechat` |  | Correctly copy the 'triggering_answer_ids' field from the original script_step_ids to the clone. This needs to be done in post-processing to make sure we get references to the newly created answers from the copy instead of references to the answers of the original.  This implementation assumes that the order of created steps and answers will be kept between the original and the clone, using 'zip()' to match the records between the two. |
| `create` | lifecycle override | self, vals_list | `im_livechat` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `im_livechat` |  |  |
| `_get_welcome_steps` | preparation rule | self | `im_livechat` |  | Returns a sub-set of script_step_ids that only contains the "welcoming steps". We consider those as all the steps the bot will say before expecting a first answer from the end user.  Example 1: - step 1 (question_selection): What do you want to do? - Create a Lead, -Create a Ticket - step 2 (text): Thank you for visiting our website! -> The welcoming steps will only contain step 1, since directly after that we expect an input from the user  Example 2: - step 1 (text): Hello! I'm a bot! - step 2 (text): I am here to help lost users. - step 3 (question_selection): What do you want to do? - Creat |
| `_post_welcome_steps` | internal rule | self, discuss_channel | `im_livechat` |  | Welcome messages are only posted after the visitor's first interaction with the chatbot. See 'chatbot.script#_get_welcome_steps()' for more details.  Side note: it is important to set the 'chatbot_current_step_id' on each iteration so that it's correctly set when going into 'discuss_channel#_message_post_after_hook()'. |
| `action_view_livechat_channels` | user action | self | `im_livechat` |  |  |
| `_to_store_defaults` | internal rule | self, target | `im_livechat` |  |  |
| `_validate_email` | internal rule | self, email_address, discuss_channel | `im_livechat` |  |  |
| `_get_chatbot_language` | preparation rule | self | `im_livechat` |  |  |
| `_compute_lead_count` | computation | self | `crm_livechat` |  |  |
| `action_view_leads` | user action | self | `crm_livechat` |  |  |
| `action_test_script` | user action | self | `website_livechat` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_question_selection` | ValidationError | Step of type 'Question' must have answers. | `im_livechat` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm_livechat.chatbot_script_view_form` | div | `im_livechat.chatbot_script_view_form` | `lead_count` | `action_view_leads` |  | `crm_livechat` |
| `im_livechat.chatbot_script_view_form` | form |  | `first_step_warning`, `livechat_channel_count`, `active`, `image_1920`, `title`, `script_step_ids` | `action_view_livechat_channels` |  | `im_livechat` |
| `im_livechat.chatbot_script_view_tree` | list |  | `title` |  |  | `im_livechat` |
| `im_livechat.chatbot_script_view_search` | search |  | `title` |  | `Archived` | `im_livechat` |
| `website_livechat.chatbot_script_view_form` | xpath | `im_livechat.chatbot_script_view_form` |  | `Test` |  | `website_livechat` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `im_livechat.chatbot_script_action` | Chatbot | list,form |  |  |  | `im_livechat` |

Machine-readable definition: `../../../schemas/data/entities/chatbot.script.json`; views: `../../../schemas/interfaces/views/chatbot.script.json`.
