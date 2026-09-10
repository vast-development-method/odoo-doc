# Mail Tracking Value (`mail.tracking.value`)

**Transport name:** `mail.tracking.value`  
**Storage name:** `mail_tracking_value`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `account`

Description: Mail Tracking Value

## Identity and behavior

- Default ordering: `id DESC`
- Display name field: `field_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `field_id` | Field | many to one | `ir.model.fields` | read only; indexed; on delete of the target: set null |
| `field_info` | Removed field information | structured document |  |  |
| `old_value_integer` | Old Value Integer | integer |  | read only |
| `old_value_float` | Old Value Float | float |  | read only |
| `old_value_char` | Old Value Char | single line text |  | read only |
| `old_value_text` | Old Value Text | multi line text |  | read only |
| `old_value_datetime` | Old Value DateTime | date and time |  | read only |
| `new_value_integer` | New Value Integer | integer |  | read only |
| `new_value_float` | New Value Float | float |  | read only |
| `new_value_char` | New Value Char | single line text |  | read only |
| `new_value_text` | New Value Text | multi line text |  | read only |
| `new_value_datetime` | New Value Datetime | date and time |  | read only |
| `currency_id` | Currency | many to one | `res.currency` | read only; on delete of the target: set null; Help: Used to display the currency when tracking monetary values |
| `mail_message_id` | Message identifier | many to one | `mail.message` | required; indexed; on delete of the target: cascade |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_filter_has_field_access` | internal rule | self, env | `mail` |  | Return the subset of self for which the user in env has access. As this model is admin-only, it is generally accessed as sudo and we need to distinguish context environment from tracking values environment.  If tracking is linked to a field, user should have access to the field. Otherwise only members of "base.group_system" can access it. |
| `_filter_free_field_access` | internal rule | self | `mail` |  | Return the subset of self which is available for all users: trackings linked to an existing field without access group. It is used notably when sending tracking summary through notifications. |
| `_create_tracking_values` | internal rule | self, initial_value, new_value, col_name, col_info, record | `mail` | model | Prepare values to create a mail.tracking.value. It prepares old and new value according to the field type.  :param initial_value: field value before the change, could be text, int,   date, datetime, ...; :param new_value: field value after the change, could be text, int,   date, datetime, ...; :param str col_name: technical field name, column name (e.g. 'user_id); :param dict col_info: result of fields_get(col_name); :param <record> record: record on which tracking is performed, used for   related computation e.g. finding currency of monetary fields;  :return: a dict values valid for 'mail.tra |
| `_create_tracking_values_property` | internal rule | self, initial_value, col_name, col_info, record | `mail` | model | Generate the values for the <mail.tracking.values> corresponding to a property. |
| `_tracking_value_format` | messaging hook | self | `mail` |  | Return structure and formatted data structure to be used by chatter to display tracking values. Order it according to asked display, aka ascending sequence (and field name).  :return: for each tracking value in self, their formatted display   values given as a dict; :rtype: list[dict] |
| `_tracking_value_format_model` | messaging hook | self, model | `mail` |  | Return structure and formatted data structure to be used by chatter to display tracking values. Order it according to asked display, aka ascending sequence (and field name).  :returns: for each tracking value in self, their formatted display   values given as a dict; :rtype: list[dict] |
| `_format_display_value` | internal rule | self, field_type, new | `mail` |  | Format value of 'mail.tracking.value', according to the field type.  :param str field_type: Odoo field type; :param bool new: if True, display the 'new' value. Otherwise display   the 'old' one. |
| `_except_audit_log` | internal rule | self | `account` | ondelete |  |
| `write` | lifecycle override | self, vals | `account` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.view_mail_tracking_value_tree` | list |  | `field_id`, `old_value_integer`, `old_value_float`, `old_value_char`, `old_value_text`, `old_value_datetime`, `new_value_integer`, `new_value_float`, `new_value_char`, `new_value_text`, `new_value_datetime`, `mail_message_id` |  |  | `mail` |
| `mail.view_mail_tracking_value_form` | form |  | `field_id`, `old_value_integer`, `old_value_float`, `old_value_char`, `old_value_text`, `old_value_datetime`, `new_value_integer`, `new_value_float`, `new_value_char`, `new_value_text`, `new_value_datetime`, `mail_message_id` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.action_view_mail_tracking_value` | Tracking Values | list,form |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.tracking.value.json`; views: `../../../schemas/interfaces/views/mail.tracking.value.json`.
