# Properties Base Definition Mixin (`properties.base.definition.mixin`)

**Transport name:** `properties.base.definition.mixin`  
**Storage name:** `properties_base_definition_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`

Description: Properties Base Definition Mixin

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `properties` | Properties | properties |  |  |
| `properties_base_definition_id` | Properties Base Definition | many to one | `properties.base.definition` | computed by rule `_compute_properties_base_definition_id` (not stored); searchable through a search rule |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_properties_base_definition_id` | computation | self | `base` |  |  |
| `_search_properties_base_definition_id` | search rule | self, operator, value | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `_field_to_sql` | internal rule | self, alias, fname, query | `base` |  |  |

Machine-readable definition: `../../../schemas/data/entities/properties.base.definition.mixin.json`.
