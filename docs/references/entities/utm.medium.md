# campaign tracking parameter Medium (`utm.medium`)

**Transport name:** `utm.medium`  
**Storage name:** `utm_medium`  
**Kind:** persistent entity (one table)  
**Defined by package:** `utm`  
**Extended by packages:** `mass_mailing`, `mass_mailing_sms`

Description: UTM Medium

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Medium Name | single line text |  | required |
| `active` | Active | boolean |  | default `True` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_name` | Constraint | `UNIQUE(name)` | The name must be unique | `utm` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `utm` | model_create_multi |  |
| `SELF_REQUIRED_UTM_MEDIUMS_REF` | operation | self | `mass_mailing_sms`, `utm` |  |  |
| `_unlink_except_utm_medium_record` | internal rule | self | `utm` | ondelete |  |
| `_fetch_or_create_utm_medium` | internal rule | self, name, module | `utm` |  |  |
| `_unlink_except_linked_mailings` | internal rule | self | `mass_mailing` | ondelete | Already handled by ondelete='restrict', but let's show a nice error message |
| `_unlink_except_utm_medium_sms` | internal rule | self | `mass_mailing_sms` | ondelete |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_utm_medium_record` | UserError | Oops, you can't delete the Medium '%s'. Doing so would be like tearing down a load-bearing wall — not the best idea. | `utm` |
| `_unlink_except_linked_mailings` | UserError | You cannot delete these UTM Mediums as they are linked to the following mailings in Mass Mailing: %(mailing_names)s | `mass_mailing` |
| `_unlink_except_utm_medium_sms` | UserError | The UTM medium '%s' cannot be deleted as it is used in some main functional flows, such as the SMS Marketing. | `mass_mailing_sms` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `base.group_user` | yes | yes | yes | no | `utm` |
| `base.group_system` | yes | yes | yes | yes | `utm` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `utm.utm_medium_view_tree` | list |  | `name`, `active` |  |  | `utm` |
| `utm.utm_medium_view_form` | form |  | `name`, `active` |  |  | `utm` |
| `utm.utm_medium_view_search` | search |  | `name` |  | `Archived` | `utm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `utm.utm_medium_action` | Mediums | list,form |  |  |  | `utm` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `hr_recruitment.menu_hr_recruitment_utm_mediums` | Mediums | `menu_hr_recruitment_utm` | `utm.utm_medium_action` | 15 | `base.group_no_one` |

Machine-readable definition: `../../../schemas/data/entities/utm.medium.json`; views: `../../../schemas/interfaces/views/utm.medium.json`.
