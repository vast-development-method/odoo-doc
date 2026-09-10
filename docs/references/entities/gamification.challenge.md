# Gamification Challenge (`gamification.challenge`)

**Transport name:** `gamification.challenge`  
**Storage name:** `gamification_challenge`  
**Kind:** persistent entity (one table)  
**Defined by package:** `gamification`  
**Extended by packages:** `survey`, `website_slides`, `website_forum`

Description: Gamification Challenge

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Default ordering: `end_date, start_date, name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (26)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Challenge Name | single line text |  | required; translatable |
| `description` | Description | multi line text |  | translatable |
| `state` | State | selection |  | required; default `draft`; changes are tracked in the message thread; not copied on duplication |
| `manager_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.uid) |
| `user_ids` | Participants | many to many | `res.users` | association table `gamification_challenge_users_rel` |
| `user_domain` | User domain | single line text |  |  |
| `user_count` | # Users | integer |  | computed by rule `_compute_user_count` (not stored) |
| `period` | Periodicity | selection |  | required; default `once`; Help: Period of automatic goal assignment. If none is selected, should be launched manually. |
| `start_date` | Start Date | date |  | Help: The day a new challenge will be automatically started. If no periodicity is set, will use this date as the goal start date. |
| `end_date` | End Date | date |  | Help: The day a new challenge will be automatically closed. If no periodicity is set, will use this date as the goal end date. |
| `invited_user_ids` | Suggest to users | many to many | `res.users` | association table `gamification_invited_user_ids_rel` |
| `line_ids` | Lines | one to many | `gamification.challenge.line` | required; inverse field `challenge_id`; Help: List of goals that will be set |
| `reward_id` | For Every Succeeding User | many to one | `gamification.badge` | indexed (btree_not_null) |
| `reward_first_id` | For 1st user | many to one | `gamification.badge` |  |
| `reward_second_id` | For 2nd user | many to one | `gamification.badge` |  |
| `reward_third_id` | For 3rd user | many to one | `gamification.badge` |  |
| `reward_failure` | Reward Bests if not Succeeded? | boolean |  |  |
| `reward_realtime` | Reward as soon as every goal is reached | boolean |  | default `True`; Help: With this option enabled, a user can receive a badge only once. The top 3 badges are still rewarded only at the end of the challenge. |
| `visibility_mode` | Display Mode | selection |  | required; default `personal` |
| `report_message_frequency` | Report Frequency | selection |  | required; default `never` |
| `report_message_group_id` | Send a copy to | many to one | `discuss.channel` | Help: Group that will receive a copy of the report in addition to the user |
| `report_template_id` | Report Template | many to one | `mail.template` | required; default computed dynamically (lambda self: self._get_report_template()) |
| `remind_update_delay` | Non-updated manual goals will be reminded after | integer |  | Help: Never reminded if no value or zero is specified. |
| `last_report_date` | Last Report Date | date |  | default computed dynamically (fields.Date.today) |
| `next_report_date` | Next Report Date | date |  | computed by rule `_get_next_report_date` and stored |
| `challenge_category` | Appears in | selection |  | required; default `hr`; on delete of the target: {"forum": "set default"}; Help: Define the visibility of the challenge through menus; extended by packages `survey`, `website_slides`, `website_forum` |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `draft` | Draft |
| `inprogress` | In Progress |
| `done` | Done |

### `period` (Periodicity)

| Value | Label |
|---|---|
| `once` | Non recurring |
| `daily` | Daily |
| `weekly` | Weekly |
| `monthly` | Monthly |
| `yearly` | Yearly |

### `visibility_mode` (Display Mode)

| Value | Label |
|---|---|
| `personal` | Individual Goals |
| `ranking` | Leader Board (Group Ranking) |

### `report_message_frequency` (Report Frequency)

| Value | Label |
|---|---|
| `never` | Never |
| `onchange` | On change |
| `daily` | Daily |
| `weekly` | Weekly |
| `monthly` | Monthly |
| `yearly` | Yearly |

### `challenge_category` (Appears in)

| Value | Label |
|---|---|
| `hr` | Human Resources / Engagement |
| `other` | Settings / Gamification Tools |
| `certification` | Certifications |
| `slides` | Website / Slides |
| `forum` | Website / Forum |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (22)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `gamification` | model |  |
| `_compute_user_count` | computation | self | `gamification` | depends: `user_ids` |  |
| `_get_next_report_date` | computation | self | `gamification` | depends: `last_report_date`, `report_message_frequency` | Return the next report date based on the last report date and report period. |
| `_get_report_template` | preparation rule | self | `gamification` |  |  |
| `create` | lifecycle override | self, vals_list | `gamification` | model_create_multi | Overwrite the create method to add the user of groups |
| `write` | lifecycle override | self, vals | `gamification` |  |  |
| `_cron_update` | background operation | self, ids, commit | `gamification` | model | Daily cron check.  - Start planned challenges (in draft and with start_date = today) - Create the missing goals (eg: modified the challenge to add lines) - Update every running challenge |
| `_update_all` | internal rule | self | `gamification` |  | Update the challenges and related goals. |
| `_get_challenger_users` | preparation rule | self, domain | `gamification` |  |  |
| `_recompute_challenge_users` | internal rule | self | `gamification` |  | Recompute the domain to add new users and remove the one no longer matching the domain |
| `action_start` | user action | self | `gamification` |  | Start a challenge |
| `action_check` | user action | self | `gamification` |  | Check a challenge  Create goals that haven't been created yet (eg: if added users) Recompute the current value for each goal related |
| `action_report_progress` | user action | self | `gamification` |  | Manual report of a goal, does not influence automatic report frequency |
| `action_view_users` | user action | self | `gamification` |  | Redirect to the participants (users) list. |
| `_generate_goals_from_challenge` | internal rule | self | `gamification` |  | Generate the goals for each line and user.  If goals already exist for this line and user, the line is skipped. This can be called after each change in the list of users or lines. :param list(int) ids: the list of challenge concerned |
| `_get_serialized_challenge_lines` | preparation rule | self, user, restrict_goals, restrict_top | `gamification` |  | Return a serialised version of the goals information if the user has not completed every goal  :param user: user retrieving progress (False if no distinction,              only for ranking challenges) :param restrict_goals: compute only the results for this subset of                        gamification.goal ids, if False retrieve every                        goal of current running challenge :param int restrict_top: for challenge lines where visibility_mode is                          ``ranking``, retrieve only the best                          ``restrict_top`` results and itself, if 0         |
| `report_progress` | operation | self, users, subset_goals | `gamification` |  | Post report about the progress of the goals  :param users: users that are concerned by the report. If False, will               send the report to every user concerned (goal users and               group that receive a copy). Only used for challenge with               a visibility mode set to 'personal'. :param subset_goals: goals to restrict the report |
| `accept_challenge` | operation | self | `gamification` |  |  |
| `discard_challenge` | operation | self | `gamification` |  | The user discard the suggested challenge |
| `_check_challenge_reward` | validation | self, force | `gamification` |  | Actions for the end of a challenge  If a reward was selected, grant it to the correct users. Rewards granted at:     - the end date for a challenge with no periodicity     - the end of a period for challenge with periodicity     - when a challenge is manually closed (if no end date, a running challenge is never rewarded) |
| `_get_topN_users` | preparation rule | self, n | `gamification` |  | Get the top N users for a defined challenge  Ranking criterias:     1. succeed every goal of the challenge     2. total completeness of each goal (can be over 100)  Only users having reached every goal of the challenge will be returned unless the challenge ``reward_failure`` is set, in which case any user may be considered.  :returns: an iterable of exactly N records, either User objects or           False if there was no user for the rank. There can be no           False between two users (if users[k] = False then           users[k+1] = False |
| `_reward_user` | internal rule | self, user, badge | `gamification` |  | Create a badge user and send the badge to him  :param user: the user to reward :param badge: the concerned badge |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | You can not reset a challenge with unfinished goals. | `gamification` |
| `_get_serialized_challenge_lines` | UserError | Retrieving progress for personal challenge without user information | `gamification` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `gamification` |
| `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `base.group_portal` | no | yes | no | no | `gamification` |
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_gamification` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.challenge_list_view` | list |  | `name`, `period`, `manager_id`, `state` |  |  | `gamification` |
| `gamification.challenge_form_view` | form |  | `state`, `user_count`, `name`, `user_domain`, `period`, `visibility_mode`, `manager_id`, `start_date`, `end_date`, `line_ids`, `sequence`, `definition_id`, `condition`, `target_goal`, `definition_full_suffix`, `description`, `reward_id`, `reward_first_id`, `reward_second_id`, `reward_third_id`, `reward_failure`, `reward_realtime`, `invited_user_ids`, `report_message_frequency`, `report_template_id`, `report_message_group_id`, `remind_update_delay`, `challenge_category` | `Start Challenge`, `Refresh Challenge`, `Send Report`, `action_view_users`, `%(goals_from_challenge_act)d` |  | `gamification` |
| `gamification.view_challenge_kanban` | kanban |  | `line_ids`, `name`, `user_count` |  |  | `gamification` |
| `gamification.challenge_search_view` | search |  | `name` |  | `Running Challenges`, `HR Challenges`, `State`, `Period` | `gamification` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `gamification.challenge_list_action` | Challenges | kanban,list |  | `{'search_default_inprogress':True, 'default_inprogress':True}` |  | `gamification` |
| `hr_gamification.challenge_list_action2` | Challenges | kanban,list,form | `[('challenge_category', '=', 'hr')]` | `{'search_default_inprogress':True, 'default_inprogress':True}` |  | `hr_gamification` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `gamification.ir_cron_check_challenge` | Gamification: Goal Challenge Check | 1 days | `_cron_update` |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `gamification.simple_report_template` | Gamification: Challenge Report |  |

Machine-readable definition: `../../../schemas/data/entities/gamification.challenge.json`; views: `../../../schemas/interfaces/views/gamification.challenge.json`.
