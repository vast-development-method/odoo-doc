# Mailing List Subscription (`mailing.subscription`)

**Transport name:** `mailing.subscription`  
**Storage name:** `mailing_subscription`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mass_mailing`

Description: Mailing List Subscription

## Identity and behavior

- Default ordering: `list_id DESC, contact_id DESC`
- Display name field: `contact_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `contact_id` | Contact | many to one | `mailing.contact` | required; on delete of the target: cascade |
| `list_id` | Mailing List | many to one | `mailing.list` | required; indexed; on delete of the target: cascade |
| `opt_out` | Opt Out | boolean |  | default ; Help: The contact has chosen not to receive mails anymore from this list |
| `opt_out_reason_id` | Reason | many to one | `mailing.subscription.optout` | on delete of the target: restrict |
| `opt_out_datetime` | Unsubscription Date | date and time |  | computed by rule `_compute_opt_out_datetime` and stored |
| `message_bounce` | Message Bounce | integer |  | related through path `contact_id.message_bounce` |
| `is_blacklisted` | Is Blacklisted | boolean |  | related through path `contact_id.is_blacklisted` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_contact_list` | Constraint | `unique (contact_id, list_id)` | A mailing contact cannot subscribe to the same mailing list multiple times. | `mass_mailing` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_opt_out_datetime` | computation | self | `mass_mailing` | depends: `opt_out` |  |
| `create` | lifecycle override | self, vals_list | `mass_mailing` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mass_mailing` |  |  |
| `open_mailing_contact` | operation | self | `mass_mailing` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_subscription_view_form` | form |  | `list_id`, `is_blacklisted`, `contact_id`, `create_date`, `opt_out_datetime`, `opt_out`, `opt_out_reason_id`, `message_bounce` |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_view_graph` | graph |  | `opt_out_datetime`, `message_bounce` |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_view_pivot` | pivot |  | `opt_out_datetime`, `list_id` |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_view_tree` | list |  | `create_date`, `contact_id`, `is_blacklisted`, `list_id`, `opt_out_datetime`, `opt_out_reason_id`, `message_bounce` |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_view_search` | search |  | `list_id`, `opt_out_reason_id`, `contact_id`, `opt_out_datetime` |  | `Subscription Date`, `Unsubscription Date`, `Unsubscription Date`, `Mailing List`, `Reason` | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_subscription_action_report_optout` | Opt-Out Report | graph,pivot,list,form | `[('opt_out', '=', True)]` | `{             'search_default_group_by_opt_out_reason_id': 1,         }` |  | `mass_mailing` |

Machine-readable definition: `../../../schemas/data/entities/mailing.subscription.json`; views: `../../../schemas/interfaces/views/mailing.subscription.json`.
