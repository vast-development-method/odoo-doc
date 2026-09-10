# Mailing List (`mailing.list`)

**Transport name:** `mailing.list`  
**Storage name:** `mailing_list`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mass_mailing`  
**Extended by packages:** `mass_mailing_sms`

Description: Mailing List

## Identity and behavior

- Default ordering: `create_date DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Mailing List | single line text |  | required |
| `active` | Active | boolean |  | default `True` |
| `contact_count` | Number of Contacts | integer |  | computed by rule `_compute_mailing_list_statistics` (not stored) |
| `contact_count_email` | Number of Emails | integer |  | computed by rule `_compute_mailing_list_statistics` (not stored) |
| `contact_count_opt_out` | Number of Opted-out | integer |  | computed by rule `_compute_mailing_list_statistics` (not stored) |
| `contact_pct_opt_out` | Percentage of Opted-out | float |  | computed by rule `_compute_mailing_list_statistics` (not stored) |
| `contact_count_blacklisted` | Number of Blacklisted | integer |  | computed by rule `_compute_mailing_list_statistics` (not stored) |
| `contact_pct_blacklisted` | Percentage of Blacklisted | float |  | computed by rule `_compute_mailing_list_statistics` (not stored) |
| `contact_pct_bounce` | Percentage of Bouncing | float |  | computed by rule `_compute_mailing_list_statistics` (not stored) |
| `contact_ids` | Mailing Lists | many to many | `mailing.contact` | not copied on duplication; association table `mailing_subscription` |
| `mailing_count` | Number of Mailing | integer |  | computed by rule `_compute_mailing_count` (not stored) |
| `mailing_ids` | Mass Mailings | many to many | `mailing.mailing` | not copied on duplication; association table `mail_mass_mailing_list_rel` |
| `subscription_ids` | Subscription Information | one to many | `mailing.subscription` | inverse field `list_id` |
| `is_public` | Show In Preferences | boolean |  | default ; Help: The mailing list can be accessible by recipients in the subscription management page to allow them to update their preferences. |
| `contact_count_sms` | text message Contacts | integer |  | computed by rule `_compute_mailing_list_statistics` (not stored) |

## Operations (23)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_mailing_count` | computation | self | `mass_mailing` | depends: `mailing_ids` |  |
| `_compute_mailing_list_statistics` | computation | self | `mass_mailing` | depends: `contact_ids` | Computes various statistics for this mailing.list that allow users to have a global idea of its quality (based on blacklist, opt-outs, ...).  As some fields depend on the value of each other (mainly percentages), we compute everything in a single method. |
| `write` | lifecycle override | self, vals | `mass_mailing` |  |  |
| `_compute_display_name` | computation | self | `mass_mailing` | depends: `contact_count` |  |
| `copy_data` | lifecycle override | self, default | `mass_mailing` |  |  |
| `action_open_import` | user action | self | `mass_mailing` |  | Open the mailing list contact import wizard. |
| `action_send_mailing` | user action | self | `mass_mailing` |  | Open the mailing form view, with the current lists set as recipients. |
| `action_view_contacts` | user action | self | `mass_mailing` |  |  |
| `action_view_contacts_email` | user action | self | `mass_mailing` |  |  |
| `action_view_mailings` | user action | self | `mass_mailing_sms`, `mass_mailing` |  |  |
| `action_view_contacts_opt_out` | user action | self | `mass_mailing` |  |  |
| `action_view_contacts_blacklisted` | user action | self | `mass_mailing` |  |  |
| `action_view_contacts_bouncing` | user action | self | `mass_mailing` |  |  |
| `action_merge` | user action | self, src_lists, archive | `mass_mailing` |  | Insert all the contact from the mailing lists 'src_lists' to the mailing list in 'self'. Possibility to archive the mailing lists 'src_lists' after the merge except the destination mailing list 'self'. |
| `_update_subscription_from_email` | internal rule | self, email, opt_out, force_message | `mass_mailing` |  | When opting-out: we have to switch opted-in subscriptions. We don't need to create subscription for other lists as opt-out = not being a member.  When opting-in: we have to switch opted-out subscriptions and create subscription for other mailing lists id they are public. Indeed a contact is opted-in when being subscribed in a mailing list.  :param str email: email address that should opt-in or opt-out from   mailing lists; :param boolean opt_out: if True, opt-out from lists given by self if   'email' is member of it. If False, opt-in in lists givben by self   and create membership if not alrea |
| `_mailing_get_default_domain` | messaging hook | self, mailing | `mass_mailing` |  |  |
| `_mailing_get_opt_out_list` | messaging hook | self, mailing | `mass_mailing` |  | Check subscription on all involved mailing lists. If user is opt_out on one list but not on another if two users with same email address, one opted in and the other one opted out, send the mail anyway. |
| `_fetch_contact_statistics` | internal rule | self | `mass_mailing` |  | Compute number of contacts matching various conditions. (see '_get_contact_count_select_fields' for details)  Will return a dict under the form: {     42: { # 42 being the mailing list ID         'contact_count': 52,         'contact_count_email': 35,         'contact_count_opt_out': 5,         'contact_count_blacklisted': 2     },     ... } |
| `_get_contact_statistics_fields` | preparation rule | self | `mass_mailing_sms`, `mass_mailing` |  | Returns fields and SQL query select path in a dictionnary. This is done to be easily overridable in subsequent modules.  - mailing_list_id             id of the associated mailing.list - contact_count:              all contacts - contact_count_email:        all valid emails - contact_count_opt_out:      all opted-out contacts - contact_count_blacklisted:  all blacklisted contacts |
| `_get_contact_statistics_joins` | preparation rule | self | `mass_mailing_sms`, `mass_mailing` |  | Extracted to be easily overridable by sub-modules (such as mass_mailing_sms). |
| `action_view_contacts_sms` | user action | self | `mass_mailing_sms` |  |  |
| `action_send_mailing_sms` | user action | self | `mass_mailing_sms` |  |  |
| `_mailing_get_opt_out_list_sms` | messaging hook | self, mailing | `mass_mailing_sms` |  | Check subscription on all involved mailing lists. If user is opt_out on one list but not on another, one opted in and the other one opted out, send mailing anyway.  :return: opt-outed record IDs :rtype: list |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | At least one of the mailing list you are trying to archive is used in an ongoing mailing campaign. | `mass_mailing` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `website.group_website_designer` | no | yes | no | no | `website_mass_mailing` |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_list_view_search` | search |  | `name`, `create_date` |  | `Archived`, `Creation Period` | `mass_mailing` |
| `mass_mailing.mailing_list_view_tree` | list |  | `name`, `is_public`, `mailing_count`, `contact_pct_bounce`, `contact_pct_opt_out`, `contact_pct_blacklisted`, `contact_count` |  |  | `mass_mailing` |
| `mass_mailing.mailing_list_view_form` | form |  | `contact_count`, `mailing_count`, `contact_pct_bounce`, `contact_pct_opt_out`, `contact_pct_blacklisted`, `name`, `active`, `is_public` | `Send Mailing`, `Import Contacts`, `action_view_contacts`, `action_view_mailings`, `action_view_contacts_bouncing`, `action_view_contacts_opt_out`, `action_view_contacts_blacklisted` |  | `mass_mailing` |
| `mass_mailing.mailing_list_view_form_simplified` | form |  | `name`, `is_public` |  |  | `mass_mailing` |
| `mass_mailing.mailing_list_view_kanban` | kanban |  | `active`, `name`, `contact_count`, `contact_count_email`, `mailing_count`, `contact_pct_bounce`, `contact_pct_opt_out`, `contact_pct_blacklisted` | `action_view_contacts`, `action_view_contacts`, `Import Contacts`, `Send Mailing` |  | `mass_mailing` |
| `mass_mailing_sms.mailing_list_view_kanban` | xpath | `mass_mailing.mailing_list_view_kanban` | `contact_count_sms` |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_list_view_form` | xpath | `mass_mailing.mailing_list_view_form` |  | `Send SMS` |  | `mass_mailing_sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.action_view_mass_mailing_lists` | Mailing Lists | kanban,list,form |  |  |  | `mass_mailing` |
| `mass_mailing_sms.mailing_list_action_sms` | Mailing Lists | kanban,list,form |  | `{'mailing_sms': True}` |  | `mass_mailing_sms` |

Machine-readable definition: `../../../schemas/data/entities/mailing.list.json`; views: `../../../schemas/interfaces/views/mailing.list.json`.
