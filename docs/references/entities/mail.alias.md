# Email Aliases (`mail.alias`)

**Transport name:** `mail.alias`  
**Storage name:** `mail_alias`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `hr`

Description: Email Aliases

## Identity and behavior

- Default ordering: `alias_model_id, alias_name`
- Display name field: `alias_name`
- Display name search fields: `["alias_name", "alias_domain"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `alias_name` | Alias Name | single line text |  | not copied on duplication; Help: The name of the email alias, e.g. 'jobs' if you want to catch emails for <jobs@example.odoo.com> |
| `alias_full_name` | Alias Email | single line text |  | computed by rule `_compute_alias_full_name` and stored; indexed (btree_not_null) |
| `alias_domain_id` | Alias Domain | many to one | `mail.alias.domain` | default computed dynamically (lambda self: self.env.company.alias_domain_id); on delete of the target: restrict |
| `alias_domain` | Alias domain name | single line text |  | related through path `alias_domain_id.name` |
| `alias_model_id` | Aliased Model | many to one | `ir.model` | required; on delete of the target: cascade; restricted by domain `[('field_id.name', '=', 'message_ids')]`; Help: The model (Odoo Document Kind) to which this alias corresponds. Any incoming email that does not reply to an existing record will cause the creation of a new record of this model (e.g. a Project Task) |
| `alias_defaults` | Default Values | multi line text |  | required; default `{}`; Help: A Python dictionary that will be evaluated to provide default values when creating new records for this alias. |
| `alias_force_thread_id` | Record Thread identifier | integer |  | Help: Optional ID of a thread (record) to which all incoming messages will be attached, even if they did not reply to it. If set, this will disable the creation of new records completely. |
| `alias_parent_model_id` | Parent Model | many to one | `ir.model` | Help: Parent model holding the alias. The model holding the alias reference is not necessarily the model given by alias_model_id (example: project (parent_model) and task (model)) |
| `alias_parent_thread_id` | Parent Record Thread identifier | integer |  | Help: ID of the parent record holding the alias (example: project holding the task creation alias) |
| `alias_contact` | Alias Contact Security | selection |  | required; default `everyone`; on delete of the target: {"employees": "cascade"}; Help: Policy to post a message on the document using the mailgateway. - everyone: everyone can post - partners: only authenticated partners - followers: only followers of the related document or members of following channels; extended by packages `hr` |
| `alias_incoming_local` | Local-part based incoming detection | boolean |  | default  |
| `alias_bounced_content` | Custom Bounced Message | rich text |  | translatable; Help: If set, this content will automatically be sent out to unauthorized users instead of the default message. |
| `alias_status` | Alias Status | selection |  | computed by rule `_compute_alias_status` and stored; Help: Alias status assessed on the last message received. |

## Selection values

### `alias_contact` (Alias Contact Security)

| Value | Label |
|---|---|
| `everyone` | Everyone |
| `partners` | Authenticated Partners |
| `followers` | Followers only |
| `employees` | Authenticated Employees |

### `alias_status` (Alias Status)

| Value | Label |
|---|---|
| `not_tested` | Not Tested |
| `valid` | Valid |
| `invalid` | Invalid |

## State fields

State machine fields of this entity: `alias_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_domain_unique` | UniqueIndex | `(alias_name, COALESCE(alias_domain_id, 0))` |  | `mail` |

## Operations (20)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_alias_domain_id_mc` | validation | self | `mail` | constrains: `alias_domain_id`, `alias_force_thread_id`, `alias_parent_model_id`, `alias_parent_thread_id`, `alias_model_id` | Check for invalid alias domains based on company configuration. When having a parent record and/or updating an existing record alias domain should match the one used on the related record. |
| `_check_alias_is_ascii` | validation | self | `mail` | constrains: `alias_name` | The local-part ("display-name" <local-part@domain>) of an address only contains limited range of ascii characters. We DO NOT allow anything else than ASCII dot-atom formed local-part. Quoted-string and internationnal characters are to be rejected. See rfc5322 sections 3.4.1 and 3.2.3 |
| `_check_alias_defaults` | validation | self | `mail` | constrains: `alias_defaults` |  |
| `_check_alias_domain_clash` | validation | self | `mail` | constrains: `alias_name`, `alias_domain_id` | Within a given alias domain, aliases should not conflict with bounce or catchall email addresses, as emails should be unique for the gateway. |
| `_compute_alias_full_name` | computation | self | `mail` | depends: `alias_domain_id.name`, `alias_name` | A bit like display_name, but without the 'inactive alias' UI display. Moreover it is stored, allowing to search on it. |
| `_compute_display_name` | computation | self | `mail` | depends: `alias_domain`, `alias_name` | Return the mail alias display alias_name, including the catchall domain if found otherwise "Inactive Alias". e.g.`jobs@mail.odoo.com` or `jobs` or 'Inactive Alias' |
| `_compute_alias_status` | computation | self | `mail` | depends: `alias_contact`, `alias_defaults`, `alias_model_id` | Reset alias_status to "not_tested" when fields, that can be the source of an error, are modified. |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi | Creates mail.alias records according to the values provided in ``vals`` but sanitize 'alias_name' by replacing certain unsafe characters; set default alias domain if not given.  :raise UserError: if given (alias_name, alias_domain_id) already exists   or if there are duplicates in given vals_list; |
| `write` | lifecycle override | self, vals | `mail` |  | Raise UserError with a meaningful message instead of letting the uniqueness constraint raise an SQL error. To check uniqueness we have to rebuild pairs of names / domains to validate, taking into account that a void alias_domain_id is acceptable (but also raises for uniqueness). |
| `_check_unique` | validation | self, alias_names, alias_domains | `mail` |  | Check unicity constraint won't be raised, otherwise raise a UserError with a complete error message. Also check unicity against alias config parameters.  :param list alias_names: a list of names (considered as sanitized   and ready to be sent to DB); :param list alias_domains: list of alias_domain records under which   the check is performed, as uniqueness is performed for given pair   (name, alias_domain); |
| `_sanitize_allowed_domains` | internal rule | self, allowed_domains | `mail` | model | When having aliases checked on email left-part only we may define an allowed list for right-part filtering, allowing more fine-grain than either alias domain, either everything. This method sanitized its value. |
| `_sanitize_alias_name` | internal rule | self, name, is_email | `mail` | model | Cleans and sanitizes the alias name. In some cases we want the alias to be a complete email instead of just a left-part (when sanitizing default.from for example). In that case we extract the right part and put it back after sanitizing the left part.  :param str name: the alias name to sanitize; :param bool is_email: whether to keep a right part, otherwise only   left part is kept;  :returns: sanitized alias name :rtype: str |
| `_is_encodable` | internal rule | self, alias_name, charset | `mail` | model | Check if alias_name is encodable. Standard charset is ascii, as UTF-8 requires a specific extension. Not recommended for outgoing aliases. 'remove_accents' is performed as sanitization process of the name will do it anyway. |
| `open_document` | operation | self | `mail` |  |  |
| `open_parent_document` | operation | self | `mail` |  |  |
| `_get_alias_bounced_body` | preparation rule | self, message_dict | `mail` |  | Get the body of the email return in case of bounced email when the alias does not accept incoming email e.g. contact is not allowed.  :param dict message_dict: dictionary holding parsed message variables  :return: HTML to use as email body |
| `_get_alias_bounced_body_fallback` | preparation rule | self, message_dict | `mail` |  | Default body of bounced emails. See '_get_alias_bounced_body' |
| `_get_alias_contact_description` | preparation rule | self | `hr`, `mail` |  |  |
| `_get_alias_invalid_body` | preparation rule | self, message_dict | `mail` |  | Get the body of the bounced email returned when the alias is incorrectly configured e.g. error in alias_defaults.  :param dict message_dict: dictionary holding parsed message variables  :return: HTML to use as email body |
| `_alias_bounce_incoming_email` | internal rule | self, message, message_dict, set_invalid | `mail` |  | Set alias status to invalid and create bounce message to the sender.  This method must be called when a message received on the alias has caused an error due to the mis-configuration of the alias.  :param EmailMessage message: email message that is invalid and is about   to bounce; :param dict message_dict: dictionary holding parsed message variables :param bool set_invalid: set alias as invalid, to be done notably if   bounce is considered as coming from a configuration error instead of   being rejected due to alias rules; |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_alias_domain_id_mc` | ValidationError | We could not create alias %(alias_name)s because domain %(alias_domain_name)s belongs to company %(alias_company_names)s while the owner document belongs to company %(company_name)s. | `mail` |
| `_check_alias_domain_id_mc` | ValidationError | We could not create alias %(alias_name)s because domain %(alias_domain_name)s belongs to company %(alias_company_names)s while the target document belongs to company %(company_name)s. | `mail` |
| `_check_alias_is_ascii` | ValidationError | You cannot use anything else than unaccented latin characters in the alias address %(alias_name)s. | `mail` |
| `_check_alias_defaults` | ValidationError | Invalid expression, it must be a literal python dictionary definition e.g. "{'field': 'value'}" | `mail` |
| `_check_alias_domain_clash` | ValidationError | Aliases %(alias_names)s is already used as bounce or catchall address. Please choose another alias. | `mail` |
| `_check_unique` | UserError | f'{msg_begin} {msg_end}' | `mail` |
| `_check_unique` | UserError | Email aliases %(alias_name)s cannot be used on several records at the same time. Please update records one by one. | `mail` |
| `_sanitize_allowed_domains` | ValidationError | Value %(allowed_domains)s for `mail.catchall.domain.allowed` cannot be validated. It should be a comma separated list of domains e.g. example.com,example.org. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `mail` |
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_alias_view_form` | form |  | `alias_status`, `alias_name`, `alias_domain_id`, `alias_status`, `alias_model_id`, `alias_force_thread_id`, `alias_defaults`, `alias_contact`, `alias_incoming_local`, `alias_parent_model_id`, `alias_parent_thread_id`, `alias_bounced_content` | `Open Document`, `open_parent_document` |  | `mail` |
| `mail.mail_alias_view_tree` | list |  | `alias_name`, `alias_domain_id`, `alias_model_id`, `alias_force_thread_id`, `alias_parent_model_id`, `alias_parent_thread_id`, `alias_defaults`, `alias_contact`, `alias_incoming_local`, `alias_status` | `Open Document`, `Open Owner` |  | `mail` |
| `mail.mail_alias_view_search` | search |  | `alias_name`, `alias_domain_id`, `alias_model_id`, `create_uid`, `alias_force_thread_id`, `alias_parent_model_id`, `alias_parent_thread_id` |  | `Active`, `Creator`, `Alias Domain`, `Document Model`, `Container Model` | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_alias_action` | Aliases |  |  | `{                     'search_default_active': True,                 }` |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.alias.json`; views: `../../../schemas/interfaces/views/mail.alias.json`.
