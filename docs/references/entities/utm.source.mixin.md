# campaign tracking parameter Source Mixin (`utm.source.mixin`)

**Transport name:** `utm.source.mixin`  
**Storage name:** `utm_source_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `utm`

Description: UTM Source Mixin

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | related through path `source_id.name` |
| `source_id` | Source | many to one | `utm.source` | required; not copied on duplication; on delete of the target: restrict |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `utm` | model |  |
| `create` | lifecycle override | self, vals_list | `utm` | model_create_multi | Create the UTM sources if necessary, generate the name based on the content in batch. |
| `write` | lifecycle override | self, vals | `utm` |  |  |
| `copy_data` | lifecycle override | self, default | `utm` |  | Increment the counter when duplicating the source. |

Machine-readable definition: `../../../schemas/data/entities/utm.source.mixin.json`.
