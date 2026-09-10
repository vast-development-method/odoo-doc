# Privacy Log (`privacy.log`)

**Transport name:** `privacy.log`  
**Storage name:** `privacy_log`  
**Kind:** persistent entity (one table)  
**Defined by package:** `privacy_lookup`

Description: Privacy Log

## Identity and behavior

- Display name field: `user_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date` | Date | date and time |  | required; default computed dynamically (fields.Datetime.now) |
| `anonymized_name` | Anonymized Name | single line text |  | required |
| `anonymized_email` | Anonymized Email | single line text |  | required |
| `user_id` | Handled By | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user) |
| `execution_details` | Execution Details | multi line text |  |  |
| `records_description` | Found Records | multi line text |  |  |
| `additional_note` | Additional Note | multi line text |  |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `privacy_lookup` | model_create_multi |  |
| `_anonymize_name` | internal rule | self, label | `privacy_lookup` |  |  |
| `_anonymize_email` | internal rule | self, label | `privacy_lookup` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `privacy_lookup` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `privacy_lookup.privacy_log_view_list` | list |  | `date`, `anonymized_name`, `anonymized_email`, `user_id`, `execution_details` |  |  | `privacy_lookup` |
| `privacy_lookup.privacy_log_view_form` | form |  | `anonymized_name`, `anonymized_email`, `date`, `user_id`, `records_description`, `execution_details`, `additional_note` |  |  | `privacy_lookup` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `privacy_lookup.privacy_log_action` | Privacy Logs | list,form |  | `{'create': False}` |  | `privacy_lookup` |
| `privacy_lookup.privacy_log_form_action` | Privacy Logs | form |  | `{'create': False}` |  | `privacy_lookup` |

Machine-readable definition: `../../../schemas/data/entities/privacy.log.json`; views: `../../../schemas/interfaces/views/privacy.log.json`.
