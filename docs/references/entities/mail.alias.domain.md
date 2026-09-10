# Email Domain (`mail.alias.domain`)

**Transport name:** `mail.alias.domain`  
**Storage name:** `mail_alias_domain`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Email Domain

## Identity and behavior

- Default ordering: `sequence ASC, id ASC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; Help: Email domain e.g. 'example.com' in 'odoo@example.com' |
| `company_ids` | Companies | one to many | `res.company` | inverse field `alias_domain_id`; Help: Companies using this domain as default for sending mails |
| `sequence` | Sequence | integer |  | default `10` |
| `bounce_alias` | Bounce Alias | single line text |  | required; default `bounce`; Help: Local-part of email used for Return-Path used when emails bounce e.g. 'bounce' in 'bounce@example.com' |
| `bounce_email` | Bounce Email | single line text |  | computed by rule `_compute_bounce_email` (not stored) |
| `catchall_alias` | Catchall Alias | single line text |  | required; default `catchall`; Help: Local-part of email used for Reply-To to catch answers e.g. 'catchall' in 'catchall@example.com' |
| `catchall_email` | Catchall Email | single line text |  | computed by rule `_compute_catchall_email` (not stored) |
| `default_from` | Default From Alias | single line text |  | default `notifications`; Help: Default from when it does not match outgoing server filters. Can be either a local-part e.g. 'notifications' either a complete email address e.g. 'notifications@example.com' to override all outgoing emails. |
| `default_from_email` | Default From | single line text |  | computed by rule `_compute_default_from_email` (not stored) |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_bounce_email_uniques` | Constraint | `UNIQUE(bounce_alias, name)` | Bounce emails should be unique | `mail` |
| `_catchall_email_uniques` | Constraint | `UNIQUE(catchall_alias, name)` | Catchall emails should be unique | `mail` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_bounce_email` | computation | self | `mail` | depends: `bounce_alias`, `name` |  |
| `_compute_catchall_email` | computation | self | `mail` | depends: `catchall_alias`, `name` |  |
| `_compute_default_from_email` | computation | self | `mail` | depends: `default_from`, `name` | Default from may be a valid complete email and not only a left-part like bounce or catchall aliases. Adding domain name should therefore be done only if necessary. |
| `_check_bounce_catchall_uniqueness` | validation | self | `mail` | constrains: `bounce_alias`, `catchall_alias` |  |
| `_check_name` | validation | self | `mail` | constrains: `name` | Should match a sanitized version of itself, otherwise raise to warn user (do not dynamically change it, would be confusing). |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi | Sanitize bounce_alias / catchall_alias / default_from |
| `write` | lifecycle override | self, vals | `mail` |  | Sanitize name / bounce_alias / catchall_alias / default_from |
| `_check_default_from_not_used_by_users` | validation | self | `mail` |  | Check that the default from is not used by a personal mail servers. |
| `_sanitize_configuration` | internal rule | self, config_values | `mail` | model | Tool sanitizing configuration values for domains |
| `_find_aliases` | internal rule | self, email_list | `mail` | model | Utility method to find both alias domains aliases (bounce, catchall or default from) and mail aliases from an email list.  :param email_list: list of normalized emails; normalization / removing     wrong emails is considered as being caller's job |
| `_migrate_icp_to_domain` | internal rule | self | `mail` | model | Compatibility layer helping going from pre-v17 ICP to alias domains. Mainly used when base mail configuration is done with 'base' module only and 'mail' is installed afterwards: configuration should not be lost (odoo.sh use case). |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_bounce_catchall_uniqueness` | ValidationError | Bounce/Catchall '%(matching_alias_name)s' is already used. Choose another alias or change it on the linked model. | `mail` |
| `_check_bounce_catchall_uniqueness` | ValidationError | Bounce alias %(bounce)s is already used for another domain with same name. Use another bounce or simply use the other alias domain. | `mail` |
| `_check_bounce_catchall_uniqueness` | ValidationError | Catchall alias %(catchall)s is already used for another domain with same name. Use another catchall or simply use the other alias domain. | `mail` |
| `_check_bounce_catchall_uniqueness` | ValidationError | Bounce/Catchall '%(matching_alias_name)s' is already used by %(document_name)s. Choose another alias or change it on the other document. | `mail` |
| `_check_name` | ValidationError | You cannot assign an empty domain name. | `mail` |
| `_check_name` | ValidationError | You cannot use anything else than unaccented latin characters in the domain name %(domain_name)s. | `mail` |
| `_check_default_from_not_used_by_users` | UserError | A personal mail server is using that address, you can not use it. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `mail` |
| `base.group_erp_manager` | yes | yes | yes | yes | `mail` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_alias_domain_view_form` | form |  | `name`, `company_ids`, `bounce_alias`, `catchall_alias`, `default_from` |  |  | `mail` |
| `mail.mail_alias_domain_view_tree` | list |  | `sequence`, `name`, `bounce_alias`, `catchall_alias`, `default_from`, `company_ids` |  |  | `mail` |
| `mail.mail_alias_domain_view_search` | search |  | `name`, `bounce_alias`, `catchall_alias`, `company_ids` |  | `Company` | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_alias_domain_action` | Alias Domains | list,form |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.alias.domain.json`; views: `../../../schemas/interfaces/views/mail.alias.domain.json`.
