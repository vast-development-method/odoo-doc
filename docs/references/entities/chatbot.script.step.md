# Chatbot Script Step (`chatbot.script.step`)

**Transport name:** `chatbot.script.step`  
**Storage name:** `chatbot_script_step`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`  
**Extended by packages:** `crm_livechat`, `website_livechat`, `website_crm_livechat`

Description: Chatbot Script Step

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_name` (not stored) |
| `message` | Message | rich text |  | translatable |
| `sequence` | Sequence | integer |  |  |
| `chatbot_script_id` | Chatbot | many to one | `chatbot.script` | required; indexed; on delete of the target: cascade |
| `step_type` | Step Type | selection |  | required; default `text`; on delete of the target: {"create_lead": "cascade", "create_lead_and_forward": "cascade"}; extended by packages `crm_livechat` |
| `answer_ids` | Answers | one to many | `chatbot.script.answer` | inverse field `script_step_id` |
| `triggering_answer_ids` | Only If | many to many | `chatbot.script.answer` | computed by rule `_compute_triggering_answer_ids` and stored; not copied on duplication; restricted by domain `[('script_step_id.sequence', '<', sequence), ('script_step_id.chatbot_script_id', '=', chatbot_script_id)]`; Help: Show this step only if all of these answers have been selected. |
| `is_forward_operator` | Is Forward Operator | boolean |  | computed by rule `_compute_is_forward_operator` (not stored) |
| `is_forward_operator_child` | Is Forward Operator Child | boolean |  | computed by rule `_compute_is_forward_operator_child` (not stored) |
| `operator_expertise_ids` | Operator Expertise | many to many | `im_livechat.expertise` | Help: When forwarding live chat conversations, the chatbot will prioritize users with matching expertise. |
| `crm_team_id` | Sales Team | many to one | `crm.team` | indexed (btree_not_null); on delete of the target: set null; Help: Used in combination with 'create_lead' step type in order to automatically assign the created lead/opportunity to the defined team |

## Selection values

### `step_type` (Step Type)

| Value | Label |
|---|---|
| `text` | Text |
| `question_selection` | Question |
| `question_email` | Email |
| `question_phone` | Phone |
| `forward_operator` | Forward to Operator |
| `free_input_single` | Free Input |
| `free_input_multi` | Free Input (Multi-Line) |
| `create_lead` | Create Lead |
| `create_lead_and_forward` | Create Lead & Forward |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_name` | computation | self | `im_livechat` | depends: `sequence`, `chatbot_script_id`; depends_context: `lang` |  |
| `_compute_triggering_answer_ids` | computation | self | `im_livechat` | depends: `sequence` |  |
| `_compute_is_forward_operator` | computation | self | `crm_livechat`, `im_livechat` | depends: `step_type` |  |
| `_compute_is_forward_operator_child` | computation | self | `im_livechat` | depends: `chatbot_script_id.script_step_ids.answer_ids`, `chatbot_script_id.script_step_ids.is_forward_operator`, `chatbot_script_id.script_step_ids.sequence`, `chatbot_script_id.script_step_ids.step_type`, `chatbot_script_id.script_step_ids.triggering_answer_ids`, `sequence`, `triggering_answer_ids` |  |
| `create` | lifecycle override | self, vals_list | `im_livechat` | model_create_multi | Ensure we correctly assign sequences when creating steps. Indeed, sequences are very important within the script, and will break the whole flow if not correctly defined.  This override will group created steps by chatbot_id and increment the sequence accordingly. It will also look for an existing step for that chatbot and resume from the highest sequence.  This cannot be done in a default_value for the sequence field as we cannot search by runbot_id. It is also safer and more efficient to do it here (we can batch everything).  It is still possible to manually pass the 'sequence' in the values, |
| `_chatbot_prepare_customer_values` | internal rule | self, discuss_channel, create_partner, update_partner | `im_livechat`, `website_livechat` |  | Common method that allows retreiving default customer values from the discuss.channel following a chatbot.script.  This method will return a dict containing the 'customer' values such as: {     'partner': The created partner (see 'create_partner') or the partner from the       environment if not public     'email': The email extracted from the discuss.channel messages       (see step_type 'question_email')     'phone': The phone extracted from the discuss.channel messages       (see step_type 'question_phone')     'description': A default description containing the "Please contact me on" and " |
| `_find_first_user_free_input` | internal rule | self, discuss_channel | `im_livechat` |  | Find the first message from the visitor responding to a free_input step. |
| `_fetch_next_step` | internal rule | self, selected_answer_ids | `im_livechat` |  | Fetch the next step depending on the user's selected answers. If a step contains multiple triggering answers from the same step the condition between them must be a 'OR'. If is contains multiple triggering answers from different steps the condition between them must be a 'AND'.  e.g:  STEP 1 : A B STEP 2 : C D STEP 3 : E STEP 4 ONLY IF A B C E  Scenario 1 (A C E):  A in (A B) -> OK C in (C)   -> OK E in (E)   -> OK  -> OK  Scenario 2 (B D E):  B in (A B) -> OK D in (C)   -> NOK E in (E)   -> OK  -> NOK |
| `_get_parent_step` | preparation rule | self, all_parent_steps | `im_livechat` |  | Returns the first preceding step that matches either the triggering answers or the possible answers the user can select |
| `_is_last_step` | internal rule | self, discuss_channel | `im_livechat` |  |  |
| `_process_answer` | background operation | self, discuss_channel, message_body | `im_livechat` |  | Method called when the user reacts to the current chatbot.script step. For most chatbot.script.step#step_types it simply returns the next chatbot.script.step of the script (see '_fetch_next_step').  Some extra processing is done for steps of type 'question_email' and 'question_phone' where we store the user raw answer (the mail message HTML body) into the chatbot.message in order to be able to recover it later (see '_chatbot_prepare_customer_values').  :param discuss_channel: :param message_body: :return: script step to display next :rtype: 'chatbot.script.step' |
| `_process_step` | background operation | self, discuss_channel | `crm_livechat`, `im_livechat` |  | When we reach a chatbot.step in the script we need to do some processing on behalf of the bot. Which is for most chatbot.script.step#step_types just posting the message field.  Some extra processing may be required for special step types such as 'forward_operator', 'create_lead', 'create_ticket' (in their related bridge modules). Those will have a dedicated processing method with specific docstrings.  Returns the mail.message posted by the chatbot's operator_partner_id. |
| `_to_store_defaults` | internal rule | self, target | `im_livechat` |  |  |
| `_chatbot_crm_prepare_lead_values` | internal rule | self, discuss_channel, description | `crm_livechat`, `website_crm_livechat` |  |  |
| `_process_step_create_lead` | background operation | self, discuss_channel | `crm_livechat` |  | When reaching a 'create_lead' step, we extract the relevant information: visitor's email, phone and conversation history to create a crm.lead.  We use the email and phone to update the environment partner's information (if not a public user) if they differ from the current values.  The whole conversation history will be saved into the lead's description for reference. This also allows having a question of type 'free_input_multi' to let the visitor explain their interest / needs before creating the lead. |
| `_process_step_create_lead_and_forward` | background operation | self, discuss_channel | `crm_livechat` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_process_answer` | ValidationError | "%s" is not a valid email. | `im_livechat` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm_livechat.chatbot_script_step_view_form` | field | `im_livechat.chatbot_script_step_view_form` | `step_type`, `crm_team_id` |  |  | `crm_livechat` |
| `im_livechat.chatbot_script_step_view_form` | form |  | `is_forward_operator_child`, `sequence`, `message`, `chatbot_script_id`, `step_type`, `operator_expertise_ids`, `triggering_answer_ids`, `display_name`, `answer_ids`, `sequence`, `display_name`, `name`, `redirect_link` |  |  | `im_livechat` |
| `im_livechat.chatbot_script_step_view_tree` | list |  | `sequence`, `message`, `step_type`, `answer_ids`, `triggering_answer_ids` |  |  | `im_livechat` |

Machine-readable definition: `../../../schemas/data/entities/chatbot.script.step.json`; views: `../../../schemas/interfaces/views/chatbot.script.step.json`.
