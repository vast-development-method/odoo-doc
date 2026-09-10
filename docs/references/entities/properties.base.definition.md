# Properties Base Definition (`properties.base.definition`)

**Transport name:** `properties.base.definition`  
**Storage name:** `properties_base_definition`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `web`

Description: Properties Base Definition

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `properties_field_id` | Properties Field | many to one | `ir.model.fields` | required; on delete of the target: cascade |
| `properties_definition` | Properties Definition | properties definition |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_properties_field_id` | Constraint | `UNIQUE(properties_field_id)` | Only one definition per properties field | `base` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `base` | depends: `properties_field_id` |  |
| `_check_properties_field_id` | validation | self | `base` | constrains: `properties_field_id` |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `_get_definition_for_property_field` | preparation rule | self, model_name, field_name | `base` |  |  |
| `_get_definition_id_for_property_field` | preparation rule | self, model_name, field_name | `base` |  |  |
| `get_properties_base_definition` | operation | self, model_name, field_name | `web` | model | Return the base properties definition if we can read the model. |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_properties_field_id` | ValidationError | The definition needs to be linked to a properties field. Those fields are not: %s. | `base` |
| `write` | AccessError | You can not change the field of a base definition | `base` |
| `get_properties_base_definition` | AccessError | You can not read that field definition. | `web` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `base` |
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| properties.base.definition: system all access | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |
| properties.base.definition: mailing user | `[Command.link(ref('mass_mailing.group_mass_mailing_user'))]` | `[('properties_field_id', '=', user.env.ref('mass_mailing.field_mailing_contact__properties').id)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/properties.base.definition.json`.
