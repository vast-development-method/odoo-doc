# Address Format (`format.address.mixin`)

**Transport name:** `format.address.mixin`  
**Storage name:** `format_address_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`

Description: Address Format

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_extract_fields_from_address` | internal rule | self, address_line | `base` |  | Extract keys from the address line. For example, if the address line is "zip: %(zip)s, city: %(city)s.", this method will return ['zip', 'city']. |
| `_view_get_address` | internal rule | self, arch | `base` |  |  |
| `_get_view_cache_key` | preparation rule | self, view_id, view_type, **options | `base` | model | The override of _get_view, using _view_get_address, changing the architecture according to the address view of the company, makes the view cache dependent on the company. Different companies could use each a different address view |
| `_get_view` | lifecycle override | self, view_id, view_type, **options | `base` | model |  |

Machine-readable definition: `../../../schemas/data/entities/format.address.mixin.json`.
