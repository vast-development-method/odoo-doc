# in-app purchase Account (`iap.account`)

**Transport name:** `iap.account`  
**Storage name:** `iap_account`  
**Kind:** persistent entity (one table)  
**Defined by package:** `iap`  
**Extended by packages:** `iap_mail`, `sms`, `l10n_in`

Description: IAP Account

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  |  |
| `service_id` | Service | many to one | `iap.service` | required |
| `service_name` | Service Name | single line text |  | related through path `service_id.technical_name` |
| `service_locked` | Service Locked | boolean |  | default  |
| `description` | Description | single line text |  | related through path `service_id.description` |
| `account_token` | Account Token | single line text |  | default computed dynamically (lambda s: uuid.uuid4().hex); not copied on duplication; visible only to groups `base.group_system`; maximum length 43; Help: Account token is your authentication key for this service. Do not share it. |
| `company_ids` | Company | many to many | `res.company` | changes are tracked in the message thread; extended by packages `iap_mail` |
| `balance` | Balance | single line text |  | read only |
| `warning_threshold` | Email Alert Threshold | float |  | changes are tracked in the message thread; extended by packages `iap_mail` |
| `warning_user_ids` | Email Alert Recipients | many to many | `res.users` | changes are tracked in the message thread; extended by packages `iap_mail` |
| `state` | State | selection |  | read only |
| `sender_name` | Sender Name | single line text |  | read only; Help: This is the name that will be displayed as the sender of the SMS. |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `banned` | Banned |
| `registered` | Registered |
| `unregistered` | Unregistered |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (20)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `validate_warning_alerts` | validation | self | `iap` | constrains: `warning_threshold`, `warning_user_ids` |  |
| `web_read` | lifecycle override | self, *args, **kwargs | `iap` |  |  |
| `write` | lifecycle override | self, vals | `iap` |  |  |
| `_get_account_information_from_iap` | preparation rule | self | `iap` |  |  |
| `_get_account_info` | preparation rule | self, account_id, balance, information | `iap`, `sms` |  |  |
| `create` | lifecycle override | self, vals_list | `iap` | model_create_multi |  |
| `get` | operation | self, service_name, force_create | `iap` | model |  |
| `get_account_id` | operation | self, service_name | `iap` | model |  |
| `get_credits_url` | operation | self, service_name, account_token | `iap` | model | Called notably by: buy more widget, partner_autocomplete, snailmail, ... |
| `_hash_iap_token` | internal rule | self, key | `iap` | model |  |
| `action_buy_credits` | user action | self | `iap` |  |  |
| `get_config_account_url` | operation | self | `iap` | model | Called notably by ajax partner_autocomplete. |
| `get_credits` | operation | self, service_name | `iap` | model |  |
| `_send_success_notification` | internal rule | self, message, title | `iap_mail` | model |  |
| `_send_error_notification` | internal rule | self, message, title | `iap_mail` | model |  |
| `_send_status_notification` | internal rule | self, message, status, title | `iap_mail` | model |  |
| `_send_no_credit_notification` | internal rule | self, service_name, title | `iap_mail` | model |  |
| `action_open_registration_wizard` | user action | self | `sms` |  |  |
| `action_open_sender_name_wizard` | user action | self | `sms` |  |  |
| `_l10n_in_connect_to_server` | internal rule | self, is_production, params, url_path, config_parameter, timeout | `l10n_in` | model |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `validate_warning_alerts` | UserError | Please set a positive email alert threshold. | `iap` |
| `validate_warning_alerts` | UserError | One of the email alert recipients doesn't have an email address set. Users: %s | `iap` |
| `get` | UserError | No service exists with the provided technical name | `iap` |
| `_hash_iap_token` | UserError | The IAP token provided is invalid or empty. | `iap` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `iap` |
| `base.group_user` | yes | yes | no | no | `iap` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| User IAP Account | `[(4, ref('base.group_user'))]` | `['\|', ('company_ids', '=', False), ('company_ids', 'in', company_ids)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `iap.iap_account_view_form` | form |  | `name`, `service_id`, `description`, `company_ids`, `account_token`, `balance`, `warning_threshold`, `warning_user_ids` | `action_buy_credits` |  | `iap` |
| `iap.iap_account_view_tree` | list |  | `name`, `service_id`, `company_ids`, `account_token`, `balance`, `warning_threshold`, `description` |  |  | `iap` |
| `iap_mail.iap_account_view_form` | xpath | `iap.iap_account_view_form` |  |  |  | `iap_mail` |
| `sms.iap_account_view_form` | xpath | `iap.iap_account_view_form` |  | `action_open_registration_wizard` |  | `sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `iap.iap_account_action` | IAP Account | list,form |  |  |  | `iap` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `crm_iap_mine.lead_generation_no_credits` | IAP Lead Generation Notification | IAP Lead Generation Notification |

Machine-readable definition: `../../../schemas/data/entities/iap.account.json`; views: `../../../schemas/interfaces/views/iap.account.json`.
