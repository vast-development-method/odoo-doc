# Email Aliases Mixin (light) (`mail.alias.mixin.optional`)

**Transport name:** `mail.alias.mixin.optional`  
**Storage name:** `mail_alias_mixin_optional`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`

Description: Email Aliases Mixin (light)

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `alias_id` | Alias | many to one | `mail.alias` | not copied on duplication; on delete of the target: restrict |
| `alias_name` | Alias Name | single line text |  | related through path `alias_id.alias_name` |
| `alias_domain_id` | Alias Domain | many to one | `mail.alias.domain` | related through path `alias_id.alias_domain_id` |
| `alias_domain` | Alias Domain Name | single line text |  | related through path `alias_id.alias_domain` |
| `alias_defaults` | Alias Defaults | multi line text |  | related through path `alias_id.alias_defaults` |
| `alias_email` | Email Alias | single line text |  | computed by rule `_compute_alias_email` (not stored); searchable through a search rule |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_alias_email` | computation | self | `mail` | depends: `alias_domain`, `alias_name` | Alias email can be used in views, as it is Falsy when having no domain or no name. Alias display name itself contains more info and cannot be used as it is in views. |
| `_search_alias_email` | search rule | self, operator, operand | `mail` |  |  |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi | Create aliases using sudo if an alias is required, notably if its name is given. |
| `write` | lifecycle override | self, vals | `mail` |  | Split writable fields of mail.alias and other fields alias fields will write with sudo and the other normally. Also handle alias_domain_id update. If alias does not exist and we try to set a name, create the alias automatically. |
| `unlink` | lifecycle override | self | `mail` |  | Delete the given records, and cascade-delete their corresponding alias. |
| `copy_data` | lifecycle override | self, default | `mail` |  |  |
| `_require_new_alias` | internal rule | self, record_vals | `mail` | model | Create only if no existing alias, and if a name is given, to avoid creating inactive aliases (falsy name). |
| `_alias_get_alias_domain_id` | internal rule | self | `mail` |  | Return alias domain value to synchronize with owner's company. Implementing it with a compute is complicated, as its 'alias_domain_id' is a field on 'mail.alias' model, coming from 'alias_id' field and due to current implementation of the mixin, notably the create / write overrides, compute is not called in all cases. We therefore use a tool method to call in the mixin. |
| `_alias_get_creation_values` | internal rule | self | `mail` |  | Return values to create an alias, or to write on the alias after its creation. |
| `_alias_filter_fields` | internal rule | self, values, filters | `mail` |  | Split the vals dict into two dictionnary of vals, one for alias field and the other for other fields |

Machine-readable definition: `../../../schemas/data/entities/mail.alias.mixin.optional.json`.
