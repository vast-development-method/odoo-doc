# Livechat Channel Rules (`im_livechat.channel.rule`)

**Transport name:** `im_livechat.channel.rule`  
**Storage name:** `im_livechat_channel_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `im_livechat`

Description: Livechat Channel Rules

## Identity and behavior

- Default ordering: `sequence asc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `regex_url` | uniform resource locator Regex | single line text |  | Help: Regular expression specifying the web pages this rule will be applied on. |
| `action` | Live Chat Button | selection |  | required; default `display_button`; Help: * 'Show' displays the chat button on the pages. * 'Show with notification' is 'Show' in addition to a floating text just next to the button. * 'Open automatically' displays the button and automatically opens the conversation pane on larger screens. On small screens, this behaves like 'Show'. * 'Hide' hides the chat button on the pages. |
| `auto_popup_timer` | Time to Open | integer |  | default ; Help: Delay (in seconds) to automatically open the conversation window. Note: the selected action must be 'Open automatically' otherwise this parameter will not be taken into account. |
| `chatbot_script_id` | Chatbot | many to one | `chatbot.script` |  |
| `chatbot_enabled_condition` | Enable ChatBot | selection |  | required; default `always` |
| `channel_id` | Channel | many to one | `im_livechat.channel` | indexed (btree_not_null); Help: The channel of the rule |
| `country_ids` | Countries | many to many | `res.country` | association table `im_livechat_channel_country_rel`; Help: The rule will only be applied for these countries. Example: if you select 'Belgium' and 'United States' and that you set the action to 'Hide', the chat button will be hidden on the specified URL from the visitors located in these 2 countries. This feature requires GeoIP installed on your server. |
| `sequence` | Matching order | integer |  | default `10`; Help: Given the order to find a matching rule. If 2 rules are matching for the given url/country, the one with the lowest sequence will be chosen. |

## Selection values

### `action` (Live Chat Button)

| Value | Label |
|---|---|
| `display_button` | Show |
| `display_button_and_text` | Show with notification |
| `auto_popup` | Open automatically |
| `hide_button` | Hide |

### `chatbot_enabled_condition` (Enable ChatBot)

| Value | Label |
|---|---|
| `always` | Always |
| `only_if_no_operator` | Only when no operator is available |
| `only_if_operator` | Only when an operator is available |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `match_rule` | operation | self, channel_id, url, country_id | `im_livechat` |  | determine if a rule of the given channel matches with the given url :param channel_id : the identifier of the channel_id :param url : the url to match with a rule :param country_id : the identifier of the country :returns the rule that matches the given condition. False otherwise. :rtype : im_livechat.channel.rule |
| `_is_bot_configured` | internal rule | self | `im_livechat` |  |  |
| `_to_store_defaults` | internal rule | self, target | `im_livechat` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `im_livechat_group_user` | yes | yes | yes | no | `im_livechat` |
| `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `im_livechat.im_livechat_channel_rule_view_tree` | list |  | `sequence`, `action`, `chatbot_script_id`, `regex_url`, `country_ids` |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_rule_view_kanban` | kanban |  | `action`, `regex_url`, `country_ids` |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_rule_view_form` | form |  | `action`, `auto_popup_timer`, `chatbot_script_id`, `chatbot_enabled_condition`, `regex_url`, `country_ids` |  |  | `im_livechat` |

Machine-readable definition: `../../../schemas/data/entities/im_livechat.channel.rule.json`; views: `../../../schemas/interfaces/views/im_livechat.channel.rule.json`.
