# Keep the call history (`discuss.call.history`)

**Transport name:** `discuss.call.history`  
**Storage name:** `discuss_call_history`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `im_livechat`

Description: Keep the call history

## Identity and behavior

- Default ordering: `start_dt DESC, id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `channel_id` | Channel | many to one | `discuss.channel` | required; indexed; on delete of the target: cascade |
| `duration_hour` | Duration Hour | float |  | computed by rule `_compute_duration_hour` (not stored) |
| `start_dt` | Start Dt | date and time |  | required; indexed |
| `end_dt` | End Dt | date and time |  |  |
| `start_call_message_id` | Start Call Message | many to one | `mail.message` | indexed |
| `livechat_participant_history_ids` | Livechat Participant History | many to many | `im_livechat.channel.member.history` |  |

## Database constraints and indexes (4)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_channel_id_not_null_constraint` | Constraint | `CHECK (channel_id IS NOT NULL)` | Call history must have a channel | `mail` |
| `_start_dt_is_not_null_constraint` | Constraint | `CHECK (start_dt IS NOT NULL)` | Call history must have a start date | `mail` |
| `_message_id_unique_constraint` | Constraint | `UNIQUE (start_call_message_id)` | Messages can only be linked to one call history | `mail` |
| `_channel_id_end_dt_idx` | Index | `(channel_id, end_dt) WHERE end_dt IS NULL` |  | `mail` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_duration_hour` | computation | self | `mail` | depends: `start_dt`, `end_dt` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `mail` |
| `base.group_public` | no | yes | no | no | `mail` |
| `base.group_portal` | no | yes | no | no | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| discuss.call.history: livechat users can access all call histories | `[(4, ref('im_livechat_group_user'))]` | `[('channel_id.channel_type', '=', 'livechat')]` | True | False | False | False |
| discuss.call.history: read call history of accessible channels | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | True | False | False | False |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.discuss_call_history_view_tree` | list |  | `channel_id`, `start_dt`, `end_dt`, `duration_hour` |  |  | `mail` |
| `mail.discuss_call_history_view_form` | form |  | `channel_id`, `start_dt`, `end_dt`, `duration_hour` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.discuss_call_history_action` | Call History | list,form |  | `{"create": False}` |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/discuss.call.history.json`; views: `../../../schemas/interfaces/views/discuss.call.history.json`.
