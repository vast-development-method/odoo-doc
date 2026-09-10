# Mailing Favorite Filters (`mailing.filter`)

**Transport name:** `mailing.filter`  
**Storage name:** `mailing_filter`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mass_mailing`

Description: Mailing Favorite Filters

## Identity and behavior

- Default ordering: `create_date DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `create_uid` | Saved by | many to one | `res.users` | read only; default computed dynamically (lambda self: self.env.user); indexed |
| `name` | Filter Name | single line text |  | required |
| `mailing_domain` | Filter Domain | single line text |  | required |
| `mailing_model_id` | Recipients Model | many to one | `ir.model` | required; on delete of the target: cascade |
| `mailing_model_name` | Recipients Model Name | single line text |  | related through path `mailing_model_id.model` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_mailing_domain` | validation | self | `mass_mailing` | constrains: `mailing_domain`, `mailing_model_id` | Check that if the mailing domain is set, it is a valid one |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_mailing_domain` | ValidationError | The filter domain is not valid for this recipients. | `mass_mailing` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_filter_view_search` | search |  | `name`, `mailing_model_id` |  | `My Filters`, `Recipients` | `mass_mailing` |
| `mass_mailing.mailing_filter_view_tree` | list |  | `name`, `create_uid`, `mailing_model_id`, `mailing_domain` |  |  | `mass_mailing` |
| `mass_mailing.mailing_filter_view_form` | form |  | `name`, `mailing_model_id`, `create_uid`, `mailing_model_name`, `mailing_domain` |  |  | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_filter_action` | Favorite Filters | list,form |  | `{'search_default_filter_saved_by_me': 1}` |  | `mass_mailing` |

Machine-readable definition: `../../../schemas/data/entities/mailing.filter.json`; views: `../../../schemas/interfaces/views/mailing.filter.json`.
