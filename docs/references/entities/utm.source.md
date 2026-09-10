# campaign tracking parameter Source (`utm.source`)

**Transport name:** `utm.source`  
**Storage name:** `utm_source`  
**Kind:** persistent entity (one table)  
**Defined by package:** `utm`  
**Extended by packages:** `hr_recruitment`, `mass_mailing`, `marketing_card`

Description: UTM Source

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Source Name | single line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_name` | Constraint | `UNIQUE(name)` | The name must be unique | `utm` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_unlink_except_referral` | internal rule | self | `utm` | ondelete |  |
| `create` | lifecycle override | self, vals_list | `utm` | model_create_multi |  |
| `_generate_name` | internal rule | self, record, content | `utm` |  | Generate the UTM source name based on the content of the source. |
| `_unlink_except_linked_recruitment_sources` | internal rule | self | `hr_recruitment` | ondelete | Already handled by ondelete='restrict', but let's show a nice error message |
| `_unlink_except_linked_mailings` | internal rule | self | `mass_mailing` | ondelete | Already handled by ondelete='restrict', but let's show a nice error message |
| `_unlink_except_utm_source_marketing_card` | internal rule | self | `marketing_card` | ondelete |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_referral` | ValidationError | You cannot delete the 'Referral' UTM source record. | `utm` |
| `_unlink_except_linked_recruitment_sources` | UserError | You cannot delete these UTM Sources as they are linked to the following recruitment sources in Recruitment: %(recruitment_sources)s | `hr_recruitment` |
| `_unlink_except_linked_mailings` | UserError | You cannot delete these UTM Sources as they are linked to the following mailings in Mass Mailing: %(mailing_names)s | `mass_mailing` |
| `_unlink_except_utm_source_marketing_card` | UserError | The UTM source '%s' cannot be deleted as it is used to promote marketing cards campaigns. | `marketing_card` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `base.group_user` | yes | yes | yes | no | `utm` |
| `base.group_system` | yes | yes | yes | yes | `utm` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `utm.utm_source_view_tree` | list |  | `name` |  |  | `utm` |
| `utm.utm_source_view_form` | form |  | `name` |  |  | `utm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `utm.utm_source_action` | Sources | list,form |  |  |  | `utm` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `hr_recruitment.menu_hr_recruitment_utm_sources` | Sources | `menu_hr_recruitment_utm` | `utm.utm_source_action` | 15 | `base.group_no_one` |

Machine-readable definition: `../../../schemas/data/entities/utm.source.json`; views: `../../../schemas/interfaces/views/utm.source.json`.
