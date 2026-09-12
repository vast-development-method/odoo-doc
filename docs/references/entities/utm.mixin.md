# campaign tracking parameter Mixin (`utm.mixin`)

**Transport name:** `utm.mixin`  
**Storage name:** `utm_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `utm`

Description: UTM Mixin

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `campaign_id` | Campaign | many to one | `utm.campaign` | indexed (btree_not_null); Help: This is a name that helps you keep track of your different campaign efforts, e.g. Fall_Drive, Christmas_Special |
| `source_id` | Source | many to one | `utm.source` | indexed (btree_not_null); Help: This is the source of the link, e.g. Search Engine, another domain, or name of email list |
| `medium_id` | Medium | many to one | `utm.medium` | indexed (btree_not_null); Help: This is the method of delivery, e.g. Postcard, Email, or Banner Ad |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `utm` | model |  |
| `tracking_fields` | operation | self | `utm` |  |  |
| `_tracking_models` | messaging hook | self | `utm` |  |  |
| `find_or_create_record` | operation | self, model_name, name | `utm` | model | Version of `_find_or_create_record` used in frontend notably in website_links. For UTM models it calls _find_or_create_record. For other models (as through inheritance custom models could be used notably in website links) it simply calls a create. In the end it relies on standard ACLs, and is mainly a wrapper for UTM models.  :return: id of newly created or found record. As the magic of call_kw     for create is not called anymore we have to manually return an id     instead of a recordset. |
| `_find_or_create_record` | internal rule | self, model_name, name | `utm` |  | Based on the model name and on the name of the record, retrieve the corresponding record or create it. |
| `_get_unique_names` | preparation rule | self, model_name, names | `utm` | model | Generate unique names for the given model.  Take a list of names and return for each names, the new names to set in the same order (with a counter added if needed).  E.G.     The name "test" already exists in database     Input: ['test', 'test [3]', 'bob', 'test', 'test']     Output: ['test [2]', 'test [3]', 'bob', 'test [4]', 'test [5]']  :param model_name: name of the model for which we will generate unique names :param names: list of names, we will ensure that each name will be unique :return: a list of new values for each name, in the same order |
| `_split_name_and_count` | internal rule | name | `utm` |  | Return the name part and the counter based on the given name.  e.g.     "Medium" -> "Medium", 1     "Medium [1234]" -> "Medium", 1234 |

Machine-readable definition: `../../../schemas/data/entities/utm.mixin.json`.
