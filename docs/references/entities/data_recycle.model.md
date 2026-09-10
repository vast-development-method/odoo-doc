# Recycling Model (`data_recycle.model`)

**Transport name:** `data_recycle.model`  
**Storage name:** `data_recycle_model`  
**Kind:** persistent entity (one table)  
**Defined by package:** `data_recycle`

Description: Recycling Model

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Name | single line text |  | required; computed by rule `_compute_name` and stored |
| `res_model_id` | Model | many to one | `ir.model` | required; on delete of the target: cascade |
| `res_model_name` | Model Name | single line text |  | read only; related through path `res_model_id.model` and stored |
| `recycle_record_ids` | Recycle Record | one to many | `data_recycle.record` | inverse field `recycle_model_id` |
| `recycle_mode` | Recycle Mode | selection |  | required; default `manual` |
| `recycle_action` | Recycle Action | selection |  | required; default `unlink` |
| `domain` | Filter | single line text |  | computed by rule `_compute_domain` and stored |
| `time_field_id` | Time Field | many to one | `ir.model.fields` | on delete of the target: cascade; restricted by domain `[('model_id', '=', res_model_id), ('ttype', 'in', ('date', 'datetime')), ('store', '=', True)]` |
| `time_field_delta` | Delta | integer |  | default `1` |
| `time_field_delta_unit` | Delta Unit | selection |  | default `months` |
| `include_archived` | Include Archived | boolean |  |  |
| `records_to_recycle_count` | Records To Recycle | integer |  | computed by rule `_compute_records_to_recycle_count` (not stored) |
| `notify_user_ids` | Notify Users | many to many | `res.users` | default computed dynamically (lambda self: self.env.user); restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('base.group_system').id)]`; Help: List of users to notify when there are new records to recycle |
| `notify_frequency` | Notify | integer |  | default `1` |
| `notify_frequency_period` | Notify Frequency Period | selection |  | default `weeks` |
| `last_notification` | Last Notification | date and time |  | read only |

## Selection values

### `recycle_mode` (Recycle Mode)

| Value | Label |
|---|---|
| `manual` | Manual |
| `automatic` | Automatic |

### `recycle_action` (Recycle Action)

| Value | Label |
|---|---|
| `archive` | Archive |
| `unlink` | Delete |

### `time_field_delta_unit` (Delta Unit)

| Value | Label |
|---|---|
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |
| `years` | Years |

### `notify_frequency_period` (Notify Frequency Period)

| Value | Label |
|---|---|
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_notif_freq` | Constraint | `CHECK(notify_frequency > 0)` | The notification frequency should be greater than 0 | `data_recycle` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_recycle_action` | validation | self | `data_recycle` | constrains: `recycle_action` |  |
| `_compute_domain` | computation | self | `data_recycle` | depends: `res_model_id` |  |
| `_compute_name` | computation | self | `data_recycle` | depends: `res_model_id` |  |
| `_compute_records_to_recycle_count` | computation | self | `data_recycle` |  |  |
| `_cron_recycle_records` | background operation | self | `data_recycle` |  |  |
| `_recycle_records` | internal rule | self, batch_commits | `data_recycle` |  |  |
| `_notify_records_to_recycle` | internal rule | self | `data_recycle` | model |  |
| `_send_notification` | internal rule | self, delta | `data_recycle` |  |  |
| `write` | lifecycle override | self, vals | `data_recycle` |  |  |
| `open_records` | operation | self | `data_recycle` |  |  |
| `action_recycle_records` | user action | self | `data_recycle` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_recycle_action` | UserError | This model doesn't manage archived records. Only deletion is possible. | `data_recycle` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `data_recycle` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `data_recycle.view_data_recycle_model_list` | list |  | `name`, `res_model_id`, `recycle_mode`, `recycle_action`, `active` |  |  | `data_recycle` |
| `data_recycle.view_data_merge_model_form` | form |  | `records_to_recycle_count`, `name`, `res_model_id`, `res_model_name`, `active`, `recycle_mode`, `recycle_action`, `include_archived`, `notify_user_ids`, `notify_frequency`, `notify_frequency_period`, `domain`, `time_field_id`, `time_field_delta`, `time_field_delta_unit` | `Run Now`, `open_records` |  | `data_recycle` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `data_recycle.action_data_recycle_config` | Recyle Records Rules | list,form | `['\|', ('active', '=', False), ('active', '=', True)]` |  |  | `data_recycle` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `data_recycle.ir_cron_clean_records` | Data Recycle: Clean Records | 1 days | `_cron_recycle_records` |  |

Machine-readable definition: `../../../schemas/data/entities/data_recycle.model.json`; views: `../../../schemas/interfaces/views/data_recycle.model.json`.
