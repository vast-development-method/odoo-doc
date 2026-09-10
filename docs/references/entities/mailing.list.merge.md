# Merge Mass Mailing List (`mailing.list.merge`)

**Transport name:** `mailing.list.merge`  
**Storage name:** `mailing_list_merge`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mass_mailing`

Description: Merge Mass Mailing List

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `src_list_ids` | Mailing Lists | many to many | `mailing.list` |  |
| `dest_list_id` | Destination Mailing List | many to one | `mailing.list` |  |
| `merge_options` | Merge Option | selection |  | required; default `new` |
| `new_list_name` | New Mailing List Name | single line text |  |  |
| `archive_src_lists` | Archive source mailing lists | boolean |  | default `True` |

## Selection values

### `merge_options` (Merge Option)

| Value | Label |
|---|---|
| `new` | Merge into a new mailing list |
| `existing` | Merge into an existing mailing list |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mass_mailing` | model |  |
| `action_mailing_lists_merge` | user action | self | `mass_mailing` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | You can only apply this action from Mailing Lists. | `mass_mailing` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | no | `mass_mailing` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_list_merge_view_form` | form |  | `merge_options`, `new_list_name`, `dest_list_id`, `archive_src_lists`, `src_list_ids`, `name`, `contact_count` | `Merge`, `Cancel` |  | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_list_merge_action` | Merge | form |  |  | new | `mass_mailing` |

Machine-readable definition: `../../../schemas/data/entities/mailing.list.merge.json`; views: `../../../schemas/interfaces/views/mailing.list.merge.json`.
