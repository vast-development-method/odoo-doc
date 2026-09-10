# Mailing Contact (`mailing.contact`)

**Transport name:** `mailing.contact`  
**Storage name:** `mailing_contact`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mass_mailing`  
**Extended by packages:** `mass_mailing_sms`

Description: Mailing Contact

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.blacklist`, `properties.base.definition.mixin`, `mail.thread.phone`
- Default ordering: `name ASC, id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_name` and stored; changes are tracked in the message thread |
| `first_name` | First Name | single line text |  |  |
| `last_name` | Last Name | single line text |  |  |
| `company_name` | Company Name | single line text |  |  |
| `email` | Email | single line text |  |  |
| `list_ids` | Mailing Lists | many to many | `mailing.list` | association table `mailing_subscription` |
| `subscription_ids` | Subscription Information | one to many | `mailing.subscription` | inverse field `contact_id` |
| `country_id` | Country | many to one | `res.country` |  |
| `tag_ids` | Tags | many to many | `res.partner.category` |  |
| `opt_out` | Opt Out | boolean |  | computed by rule `_compute_opt_out` (not stored); searchable through a search rule; Help: Opt out flag for a specific mailing list. This field should not be used in a view without a unique and active mailing list context. |
| `mobile` | Mobile | single line text |  |  |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mass_mailing` | model | When coming from a mailing list we may have a default_list_ids context key. We should use it to create subscription_ids default value that are displayed to the user as list_ids is not displayed on form view. |
| `fields_get` | lifecycle override | self, allfields, attributes | `mass_mailing` | model | Hide first and last name field if the split name feature is not enabled. |
| `_search_opt_out` | search rule | self, operator, value | `mass_mailing` | model |  |
| `_compute_name` | computation | self | `mass_mailing` | depends: `first_name`, `last_name` |  |
| `_compute_opt_out` | computation | self | `mass_mailing` | depends: `subscription_ids`; depends_context: `default_list_ids` |  |
| `create` | lifecycle override | self, vals_list | `mass_mailing` | model_create_multi | Synchronize default_list_ids (currently used notably for computed fields) default key with subscription_ids given by user when creating contacts.  Those two values have the same purpose, adding a list to to the contact either through a direct write on m2m, either through a write on middle model subscription.  This is a bit hackish but is due to default_list_ids key being used to compute oupt_out field. This should be cleaned in master but here we simply try to limit issues while keeping current behavior. |
| `copy` | lifecycle override | self, default | `mass_mailing` |  | Cleans the default_list_ids while duplicating mailing contact in context of a mailing list because we already have subscription lists copied over for newly created contact, no need to add the ones from default_list_ids again |
| `name_create` | lifecycle override | self, name | `mass_mailing` | model |  |
| `add_to_list` | operation | self, name, list_id | `mass_mailing` | model |  |
| `action_import` | user action | self | `mass_mailing` |  |  |
| `action_add_to_mailing_list` | user action | self | `mass_mailing` |  |  |
| `get_import_templates` | operation | self | `mass_mailing` | model |  |
| `_is_name_split_activated` | internal rule | self | `mass_mailing` | model | Return whether the contact names are populated as first and last name or as a single field (name). |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `create` | UserError | You should give either list_ids, either subscription_ids to create new contacts. | `mass_mailing` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (12)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_contact_view_search` | search |  | `name`, `tag_ids`, `list_ids`, `properties` |  | `Valid Email Recipients`, `Bounced`, `Blacklisted`, `Opted-out`, `Exclude Blacklisted Emails`, `Exclude Opt Out`, `Creation Date`, `Properties` | `mass_mailing` |
| `mass_mailing.mailing_contact_view_tree` | list |  | `create_date`, `name`, `company_name`, `email`, `is_blacklisted`, `country_id`, `message_bounce`, `opt_out`, `list_ids`, `properties` | `Import`, `Add to List` |  | `mass_mailing` |
| `mass_mailing.mailing_contact_view_kanban` | kanban |  | `name`, `message_bounce`, `tag_ids`, `email`, `company_name`, `properties` | `Import` |  | `mass_mailing` |
| `mass_mailing.mailing_contact_view_form` | form |  | `id`, `name`, `email`, `is_blacklisted`, `company_name`, `country_id`, `create_date`, `message_bounce`, `tag_ids`, `properties`, `subscription_ids`, `list_id`, `opt_out_datetime`, `opt_out`, `opt_out_reason_id`, `create_date` | `mail_action_blacklist_remove` |  | `mass_mailing` |
| `mass_mailing.mailing_contact_view_pivot` | pivot |  | `create_date` |  |  | `mass_mailing` |
| `mass_mailing.mailing_contact_view_graph` | graph |  | `create_date` |  |  | `mass_mailing` |
| `mass_mailing.mailing_contact_view_tree_split_name` | xpath | `mass_mailing.mailing_contact_view_tree` | `first_name`, `last_name` |  |  | `mass_mailing` |
| `mass_mailing.mailing_contact_view_form_split_name` | xpath | `mass_mailing.mailing_contact_view_form` |  |  |  | `mass_mailing` |
| `mass_mailing_sms.mailing_contact_view_search` | xpath | `mass_mailing.mailing_contact_view_search` | `mobile`, `phone_sanitized` |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_contact_view_tree` | xpath | `mass_mailing.mailing_contact_view_tree` | `mobile`, `phone_sanitized`, `phone_sanitized_blacklisted` |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_contact_view_form` | xpath | `mass_mailing.mailing_contact_view_form` | `mobile`, `phone_sanitized` | `phone_action_blacklist_remove` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_contact_view_kanban` | xpath | `mass_mailing.mailing_contact_view_kanban` | `mobile` |  |  | `mass_mailing_sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.action_view_mass_mailing_contacts` | Mailing List Contacts | list,kanban,form,graph,pivot |  | `{'search_default_filter_not_email_bl': 1}` |  | `mass_mailing` |
| `mass_mailing_sms.mailing_contact_action_sms` | Mailing List Contacts | list,form |  | `{'mailing_sms': True, 'search_default_filter_not_phone_bl': 1, }` |  | `mass_mailing_sms` |

Machine-readable definition: `../../../schemas/data/entities/mailing.contact.json`; views: `../../../schemas/interfaces/views/mailing.contact.json`.
