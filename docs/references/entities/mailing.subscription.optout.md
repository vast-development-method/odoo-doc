# Mailing Subscription Reason (`mailing.subscription.optout`)

**Transport name:** `mailing.subscription.optout`  
**Storage name:** `mailing_subscription_optout`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mass_mailing`

Description: Mailing Subscription Reason

## Identity and behavior

- Default ordering: `sequence ASC, create_date DESC, id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reason | single line text |  | translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `is_feedback` | Ask For Feedback | boolean |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_subscription_optout_view_form` | form |  | `name`, `is_feedback` |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_optout_view_tree` | list |  | `sequence`, `name`, `is_feedback` |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_optout_view_search` | search |  | `name` |  |  | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_subscription_optout_action` | Optout Reasons | list,form |  |  |  | `mass_mailing` |

Machine-readable definition: `../../../schemas/data/entities/mailing.subscription.optout.json`; views: `../../../schemas/interfaces/views/mailing.subscription.optout.json`.
